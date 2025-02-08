## DevSecOps CI/CD Pipeline Demo

### 🛠 1. Setup & Pre-requisites
 
#### 👉 Ensure the following are ready before the demo:

Jenkins installed & running on JENKINS_HOST

SonarQube server running on SONAR_HOST

DefectDojo instance running on DEFECTDOJO_HOST

Docker & Trivy installed on Jenkins

OWASP Dependency Check & ZAP installed

GitHub repository connected

### 🚀 2. Demo

#### 💡 Step 1: Purpose of the Pipeline

**This pipeline automates security checks and deployment in a DevSecOps workflow.**

"It integrates SAST (Static Analysis), Dependency Scanning, SCA (Software Composition Analysis), DAST (Dynamic Testing), and Container Security."

### 👨‍💻 3. Walk the Pipeline

#### ✅ Step 2: Trigger the Pipeline in Jenkins

Manually Start the Pipeline

Open Jenkins → Click on Build Now

Show the logs in Jenkins Console Output

Explain what is happening at each stage.

### 🛠 4. Demonstrate Each Stage

#### 🎯 Stage 1: Code Checkout

"Jenkins pulls the latest code from GitHub."

#### 🔎 Stage 2: Static Application Security Testing (SAST) - SonarQube

Open SonarQube Web UI (http://SONAR_HOST:9000)

Show the scanned vulnerabilities & code quality reports.

#### 📝 Stage 3: Quality Gate

"Pipeline waits for the SonarQube Quality Gate result."

Show what happens if the Quality Gate fails.

#### 🛡️ Stage 4: Dependency Scanning - OWASP Dependency Check

"This stage checks for vulnerable dependencies in the code."

Show the Dependency Check report.

#### 🛡️ Stage 5: File System Security Scan - Trivy

"Trivy scans the codebase for known vulnerabilities."

Show the Trivy Report in trivy-fs-report.json.

#### 🛠️ Stage 6: Build & Deploy Docker Image

Show how Jenkins builds a secure Docker image.

Explain how it's tagged & pushed to DockerHub.

#### 🐳 Stage 7: Container Security Scan - Trivy

"Trivy scans the Docker image for vulnerabilities before deployment."

Show the Trivy Image Report.

####  Stage 8: Push Image to DockerHub

Open DockerHub and show the latest pushed image.

#### 🧕 Stage 9: Dynamic Application Security Testing (DAST) - OWASP ZAP

Explain how the app is deployed to TARGET_HOST.

Show OWASP ZAP scanning logs.

Open DefectDojo and show imported ZAP scan results.

####  Stage 10: Import Security Reports to DefectDojo

Open DefectDojo Web UI (http://DEFECTDOJO_HOST:8080)

Show Trivy, ZAP, SonarQube & Dependency Check reports.

### 🎯 5. Show the Final Deployment

Open the running application (http://TARGET_HOST:8080/WebGoat)

Show that the pipeline successfully deployed the application.

Explain how security checks prevent insecure code from being deployed.

### 🎤 6. Wrap Up - Key Takeaways

"This pipeline integrates security into DevOps."

"Automates SAST, DAST, dependency scanning, and container security."

"Ensures only secure, high-quality code reaches production."

   
