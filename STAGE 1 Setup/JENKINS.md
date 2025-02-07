## Installing Jenkins on EC2 instance
 

```bash
#!/bin/bash

# Install OpenJDK 17 JRE Headless
sudo apt install openjdk-17-jre-headless -y

# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y
```

Save this script in a file, for example, `install_jenkins.sh`, and make it executable using:

```bash
chmod +x install_jenkins.sh
```

Then, you can run the script using:

```bash
./install_jenkins.sh
```

## 🔑 Get the Jenkins Admin Password  

Jenkins requires an initial password to set up:  

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```
## 🌐 Access Jenkins Web UI  

Now, open your browser and visit:  

http://your-ec2-public-ip:8080


🔹 Enter the password stored here: /var/lib/jenkins/secrets/initialAdminPassword.  
🔹 Follow the setup wizard (Install suggested plugins, create an admin user, etc.).  


## 🚀 Installing Required Plugins on Jenkins  


1️⃣ **Login to Jenkins Web UI**  
   - Open your browser and visit:  
     ```
     http://your-ec2-public-ip:8080
     ```
   - Log in with your **admin credentials**.  

2️⃣ **Navigate to Plugin Manager**  
   - Click on **Manage Jenkins** from the left sidebar.  
   - Select **Manage Plugins**.  
   - Go to the **Available** tab (If the plugin is already installed, check the **Installed** tab).  

3️⃣ **Search and Install the Required Plugins**  
   - Use the **Search box** to find each of the following plugins and check the box:  
     
     🔹 **Docker Pipeline Plugin**  
     🔹 **SonarQube Plugin**  
     🔹 **OWASP Markup Formatter Plugin**  
     🔹 **OWASP Dependency-Check Plugin**  
     🔹 **Prometheus Metrics Plugin**  
     🔹 **Quality Gates Plugin**  
     🔹 **Sonar Quality Gates Plugin**  
     🔹 **SonarQube Scanner for Jenkins**  

4️⃣ **Install the Plugins**  
   - Click **Install without restart** to install them immediately.  
   - Alternatively, select **Download now and install after restart** if you plan to restart Jenkins later.  

5️⃣ **Verify Plugin Installation**  
   - After installation, go to **Manage Jenkins** → **Manage Plugins** → **Installed** tab.  
   - Ensure all required plugins appear in the **Installed Plugins** list.  

6️⃣ **Restart Jenkins (If Needed)**  
   - If some plugins require a restart, navigate to:  
     ```
     http://your-jenkins-url:8080/restart
     ```
   - Click **Yes** to restart Jenkins.  

---

## 🔑 Adding Credentials in Jenkins  


1️⃣ **Login to Jenkins Web UI**  
   - Open your browser and visit:  
     ```
     http://your-ec2-public-ip:8080
     ```
   - Log in with your **admin credentials**.  

2️⃣ **Navigate to Credentials Manager**  
   - Click on **Dashboard** → **Manage Jenkins** → **Credentials**.  

3️⃣ **Select the Credentials Store**  
   - Click on **Jenkins Credentials Provider**.  
   - Select **System** (global credentials).  

4️⃣ **Add New Credentials**  
   - Click **(+) Add Credentials**.  
   - Fill in the following details for each type of credential:  

---

### 🔹 **Adding SonarQube Token (Secret Text)**  
   - **Kind:** `Secret text`  
   - **Scope:** `Global`  
   - **Secret:** **(http://your-sonarqube-url/admin/webhooks ) -> create**
   - **ID:** `Sonar`  
   - **Description:** `SonarQube API Token`  
   - Click **OK**.  

---

### 🔹 **Adding GitHub API Key (Secret Text)**  
   - **Kind:** `Secret text`  
   - **Scope:** `Global`  
   - **Secret:** *(Paste GitHub API Key)*  
   - **ID:** `github-api-key`  
   - **Description:** `GitHub API Key`  
   - Click **OK**.  

---

### 🔹 **Adding Docker Credentials (Username with Password)**  
   - **Kind:** `Username with password`  
   - **Scope:** `Global`  
   - **Username:** *(Your Docker Hub username)*  
   - **Password:** *(Your Docker Hub password)*  
   - **ID:** `docker-cred`  
   - **Description:** `Docker Credentials`  
   - Click **OK**.  

---

### 🔹 **Adding SSH Public Key**  
   - **Kind:** `SSH Username with private key`  
   - **Scope:** `Global`  
   - **Username:** *(Your SSH username, e.g., root)*  
   - **Private Key:** *(Paste your SSH private key or use an existing key file)*  
   - **ID:** `ssh-cred`  
   - **Description:** `SSH Public Key Authentication`  
   - Click **OK**.  

---

### 🔹 **Adding DefectDojo API Key (Secret Text)**  
   - **Kind:** `Secret text`  
   - **Scope:** `Global`  
   - **Secret:** *(Paste DefectDojo API Key)*  
   - **ID:** `defectdojo-api-key`  
   - **Description:** `DefectDojo API Key`  
   - Click **OK**.  

---

## ✅ **Verify Added Credentials**  
Once all credentials are added, navigate back to **Manage Jenkins** → **Credentials** to confirm the following entries exist:  

| **Type**               | **ID**                  | **Description**            |
|------------------------|------------------------|----------------------------|
| Secret Text           | `Sonar`                | SonarQube API Token       |
| Secret Text           | `github-api-key`       | GitHub API Key            |
| Username with Password | `docker-cred`          | Docker Credentials        |
| SSH Private Key       | `ssh-cred`             | SSH Public Key Authentication |
| Secret Text           | `defectdojo-api-key`    | DefectDojo API Key        |

---
## 🚀 Installing Trivy on Jenkins Machine  

Run the following commands to install **Trivy**:  

```bash
sudo apt-get install -y wget apt-transport-https gnupg lsb-release && \
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add - && \
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list && \
sudo apt-get update && \
sudo apt-get install -y trivy
```
To check version:

``` 
trivy --version
```





 

