## Pipeline
```groovy
pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        TARGET_HOST = '<TARGET_HOST_IP>'  // Replace with your actual target host IP
        DEFECTDOJO_HOST = '<DEFECTDOJO_HOST_IP>'  // Replace with your actual DefectDojo host IP
        SONAR_HOST = '<SONAR_HOST_IP>'  // Replace with your actual SonarQube host IP
        DEFECTDOJO_API_KEY = credentials('defectdojo-api-key')
        JENKINS_HOST = '<JENKINS_HOST_IP>'  // Replace with your actual Jenkins host IP
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
                        -Dsonar.host.url=http://<SONAR_HOST_IP>:9000/ \
                        -X
                    '''
                }
            }
            
        }
        stage('Import SonarQube Results to DefectDojo') {
                steps {
                    script {
                            def workspace = sh(script: 'pwd', returnStdout: true).trim()
            
                            sh """
                            curl -s 'http://${SONAR_HOST}:9000/api/issues/search?componentKeys=webgoat' \
                            -u admin:Password@1 > sonarqube_results.json
                
                            echo "Verifying SonarQube results file:"
                            
                            echo "SonarQube results file verification complete"
                        """

                        writeFile file: 'sonar_config.json', text: """
                        {
                            "url": "http://${SONAR_HOST}:9000",
                            "username": "admin",
                            "password": "Password@1"
                        }
                        """
            
                            
                            sh """
                            TODAY=\$(date +"%Y-%m-%d")
                            echo "Uploading scan results to DefectDojo..."
                
                            curl -X POST "http://${DEFECTDOJO_HOST}:8080/api/v2/import-scan/" \\
                            -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \\
                            -H "Accept: application/json" \\
                            -F "scan_date=\${TODAY}" \\
                            -F "minimum_severity=Low" \\
                            -F "active=true" \\
                            -F "verified=false" \\
                            -F "scan_type=SonarQube Scan" \\
                            -F "engagement=1" \\
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
        stage("OWASP Dependency Check"){
	        steps{
		            dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP-DC'
		            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'	
    	    }
        }
        stage("Trivy file system scan"){
	        steps{
		            sh 'trivy fs --format json -o trivy-fs-report.json .'
		            sh """
		        
                    TODAY=\$(date +"%Y-%m-%d")
                    echo "Uploading scan results to DefectDojo..."
                
                    curl -X POST "http://${DEFECTDOJO_HOST}:8080/api/v2/import-scan/" \\
                    -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \\
                    -H "Accept: application/json" \\
                    -F "scan_date=\${TODAY}" \\
                    -F "minimum_severity=Low" \\
                    -F "active=true" \\
                    -F "verified=false" \\
                    -F "scan_type=Trivy Scan" \\
                    -F "engagement=1" \\
                    -F "file=@${workspace}/trivy-fs-report.json" \\
                    -v
                    """
	        }
        }
        stage('Build and deploy'){
	        steps{
		        script{
			            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){
				        sh 'docker build -t bhavinprasad/webgoat:latest .'
                    }
	            }
            }
        }
        stage("Trivy docker image scan"){
	        steps{
		                sh 'trivy image --format json -o trivy-image-report.json bhavinprasad/webgoat:latest'
	        }
        }
        stage('Deploy dockerhub'){
            steps{
                script{
			            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){
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
                            ssh -o StrictHostKeyChecking=no -i \${SSH_KEY} root@<TARGET_HOST_IP> '
                        # Run ZAP scan
                        docker run --rm  -v /home/ubuntu/zap_report:/zap/wrk:rw zaproxy/zap-stable zap-full-scan.py  \
                            -t http://<TARGET_HOST_IP>:8080/WebGoat \
                            -J zap_report.json

                        scp /home/ubuntu/zap_report/zap_report.json root@${JENKINS_HOST}:${workspace}/zap_report.json

                        echo "ZAP scan completed. Report generated as zap_report.json"
                        '
                        
                        """
                        
                        sh """
                        TODAY=\$(date +"%Y-%m-%d")
                        curl -X POST "http://${DEFECTDOJO_HOST}:8080/api/v2/import-scan/" \\
                            -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \\
                            -H "Accept: application/json" \\
                            -F "scan_date=\${TODAY}" \\
                            -F "minimum_severity=Low" \\
                            -F "active=true" \\
                            -F "verified=false" \\
                            -F "scan_type=ZAP Scan" \\
                            -F "engagement=1" \\
                            -F "file=@/var/lib/jenkins/workspace/devsecops_pipeline/zap_report.json"
                        """
                        
                    }
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
