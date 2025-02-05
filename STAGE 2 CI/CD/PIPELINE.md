## Install Plugins in Jenkins

1. **SonarQube Scanner**:
   - SonarQube is a code quality and security analysis tool.
   - This plugin integrates Jenkins with SonarQube by providing a scanner that analyzes code during builds.
   - You can install it from the Jenkins plugin manager as described above.

2. **Docker**:
   - This plugin allows Jenkins to interact with Docker, enabling Docker builds and integration with Docker registries.
   - You can use it to build Docker images, run Docker containers, and push/pull images from Docker registries.
   - Install it from the plugin manager.

3. **Docker Pipeline Step**:
   - This plugin extends Jenkins Pipeline with steps to build, publish, and run Docker containers as part of your Pipeline scripts.
   - It provides a convenient way to manage Docker containers directly from Jenkins Pipelines.
   - Install it through the plugin manager like the others.

## Configure Above Plugins in Jenkins
```groovy
pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        TARGET_HOST = [TARGET_HOST_IP]
        DEFECTDOJO_HOST = [DEFECTDOJO_HOST_IP]
        SONAR_HOST = [SONAR_HOST_IP]
        DEFECTDOJO_API_KEY = credentials('defectdojo-api-key')
        SONAR_URL = "http://${SONAR_HOST}:9000"
        DEFECTDOJO_URL = "http://${DEFECTDOJO_HOST}:8080"
        ZAP_HOST = [ZAP_HOST_IP]  // Define the ZAP host IP
        WEBGOAT_URL = "http://${ZAP_HOST}:8080/WebGoat"  // Define WebGoat URL for ZAP scan
    }
    
    stages {
        stage("Clone") {
            steps {
                git branch: 'main', 
                    credentialsId: 'git-cred', 
                    url: 'https://github.com/bhavinprasad/webgoat.git'
            }
        }

        stage('SAST - SonarQube') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=webgoat \
                        -Dsonar.projectName=webgoat \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.language=java \
                        -Dsonar.java.source=1.8 \
                        -Dsonar.host.url=${SONAR_URL} \
                        -X
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    try {
                        timeout(time: 5, unit: 'MINUTES') {
                            def qg = waitForQualityGate(abortPipeline: false)
                            if (qg.status != 'OK') {
                                echo "Quality Gate failed: ${qg.status}"
                            }
                        }
                    } catch (Exception e) {
                        echo "Quality Gate check failed: ${e.message}"
                    }
                }
            }
        }
        
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP-DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'	
            }
        }

        stage("Trivy file system scan") {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
                sh """
                    TODAY=\$(date +"%Y-%m-%d")
                    echo "Uploading scan results to DefectDojo..."
                    
                    curl -X POST "${DEFECTDOJO_URL}/api/v2/import-scan/" \\
                        -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \\
                        -H "Accept: application/json" \\
                        -F "scan_date=\${TODAY}" \\
                        -F "minimum_severity=Low" \\
                        -F "active=true" \\
                        -F "verified=false" \\
                        -F "scan_type=Trivy Scan" \\
                        -F "engagement=2" \\
                        -F "file=@${workspace}/trivy-fs-report.html" \\
                        -v
                """
            }
        }

        stage('Build and deploy') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t bhavinprasad/webgoat:latest .'
                    }
                }
            }
        }

        stage("Trivy docker image scan") {
            steps {
                sh 'trivy image --format json -o trivy-image-report.json bhavinprasad/webgoat:latest '
                sh """
                    TODAY=\$(date +"%Y-%m-%d")
                    echo "Uploading scan results to DefectDojo..."
                    
                    curl -X POST "${DEFECTDOJO_URL}/api/v2/import-scan/" \\
                        -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \\
                        -H "Accept: application/json" \\
                        -F "scan_date=\${TODAY}" \\
                        -F "minimum_severity=Low" \\
                        -F "active=true" \\
                        -F "verified=false" \\
                        -F "scan_type=Trivy JSON Scan" \\
                        -F "engagement=2" \\
                        -F "file=@${workspace}/trivy-image-report.json" \\
                        -v
                """
            }
        }

        stage('Deploy dockerhub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push bhavinprasad/webgoat:latest"
                    }
                }
            }
        }

        stage('Dynamic analysis') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh-cred', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \${SSH_KEY} root@3.91.70.18 '
                            docker run --rm -v /home/ubuntu/zap_report:/zap/wrk:rw zaproxy/zap-stable zap-full-scan.py  \
                                -t ${WEBGOAT_URL} \
                                -r zap_report.html || true
                            
                            echo "ZAP scan completed. Report generated as zap_report.html"
                        '
                    }
                }
            }
        }

        stage('Import SonarQube Results to DefectDojo') {
            steps {
                script {
                    def workspace = sh(script: 'pwd', returnStdout: true).trim()
                    
                    sh """
                        curl -s '${SONAR_URL}/api/issues/search?componentKeys=webgoat' \
                            -u admin:Password@1 > sonarqube_results.json
                        
                        echo "Verifying SonarQube results file:"
                        cat sonarqube_results.json
                        echo "SonarQube results file verification complete"
                    """
                    
                    writeFile file: 'sonar_config.json', text: """
                    {
                        "url": "${SONAR_URL}",
                        "username": "admin",
                        "password": "Password@1"
                    }
                    """
                    
                    sh "cat sonar_config.json"
                    
                    sh """
                        TODAY=\$(date +"%Y-%m-%d")
                        echo "Uploading scan results to DefectDojo..."
                        
                        curl -X POST "${DEFECTDOJO_URL}/api/v2/import-scan/" \\
                            -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \\
                            -H "Accept: application/json" \\
                            -F "scan_date=\${TODAY}" \\
                            -F "minimum_severity=Low" \\
                            -F "active=true" \\
                            -F "verified=false" \\
                            -F "scan_type=SonarQube Scan" \\
                            -F "engagement=2" \\
                            -F "file=@${workspace}/sonarqube_results.json" \\
                            -F "sonar_config=@${workspace}/sonar_config.json" \\
                            -v
                    """

                    sh '''
                        echo "Cleaning up temporary files..."
                        rm -f sonarqube_results.json sonar_config.json
                        echo "Cleanup complete"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline"
        }
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}

```