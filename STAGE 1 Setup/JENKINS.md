## Installing Jenkins on Ubuntu instance


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

