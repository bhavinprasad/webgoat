# SonarQube Setup Guide on AWS EC2 Instance

The steps include creating the EC2 instance, SSH-ing into it, and configuring SonarQube in a Docker container.

## Step 1: Create an AWS EC2 Instance

1. **Log in to AWS Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/).
   - Navigate to **EC2** and click on **Launch Instance**.

2. **Select an AMI**:
   - Choose an **Ubuntu Server** AMI (e.g., **Ubuntu 22.04 LTS**).

3. **Choose Instance Type**:
   - Select an instance type (e.g., **t2.medium** for a basic setup or choose one based on your requirements).

4. **Configure Instance Details**:
   - You can leave most settings at their defaults.
   - Ensure that your instance has a **public IP** assigned (make sure to select "Auto-assign Public IP").

5. **Add Storage**:
   - The default storage size is typically enough, but you can adjust it if necessary.

6. **Configure Security Group**:
   - Create a new security group or choose an existing one.
   - Allow incoming traffic on the following ports:
     - **SSH (Port 22)** for remote access.
     - **HTTP (Port 80)** if you want web access for other services (optional).
     - **SonarQube (Port 9000)** for SonarQube web access.

7. **Launch Instance**:
   - Review your configuration and click **Launch**.
   - Select an existing **Key Pair** or create a new one to access the instance via SSH.

8. **Connect to EC2 Instance**:
   - Once your EC2 instance is running, note its **public IP**.
   - Use SSH to connect to the instance:
     ```bash
     ssh -i /path/to/your-key-pair.pem ubuntu@<your-ec2-public-ip>
     ```

---

## Step 2: Install Docker on the EC2 Instance

Once connected to the EC2 instance via SSH, run the following script to install **Docker**.

# SetUp SonarQube

```bash
#!/bin/bash

# Update package manager repositories
sudo apt-get update

# Install necessary dependencies
sudo apt-get install -y ca-certificates curl

# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package manager repositories
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
```

Save this script in a file, for example, `install_docker.sh`, and make it executable using:

```bash
chmod +x install_docker.sh
```

Then, you can run the script using:

```bash
./install_docker.sh
```

## Create Sonarqube Docker container
To run SonarQube in a Docker container with the provided command, you can follow these steps:

1. Open your terminal or command prompt.

2. Run the following command:

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

This command will download the `sonarqube:lts-community` Docker image from Docker Hub if it's not already available locally. Then, it will create a container named "sonar" from this image, running it in detached mode (`-d` flag) and mapping port 9000 on the host machine to port 9000 in the container (`-p 9000:9000` flag).

3. Access SonarQube by opening a web browser and navigating to `http://VmIP:9000`.

This will start the SonarQube server, and you should be able to access it using the provided URL. If you're running Docker on a remote server or a different port, replace `localhost` with the appropriate hostname or IP address and adjust the port accordingly.

 docker-buildx-plugin docker-compose-plugin 
