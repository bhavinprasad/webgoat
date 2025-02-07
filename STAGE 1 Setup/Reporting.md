# 🚀 Setting Up DefectDojo on an AWS EC2 Instance  

This guide provides step-by-step instructions to set up **DefectDojo**, a security vulnerability management tool, on an **AWS EC2 instance** using Docker.

---

## 📌 Prerequisites  

Before proceeding, ensure you have:  
- An **AWS EC2 Ubuntu instance**  
- **Docker** and **Docker Compose** installed  
- **Sudo privileges** on the machine  

---

## 🔑 **Step 1: Connect to the EC2 Instance**  

Use the following command to SSH into the EC2 instance:  

```bash
ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip
```
## 🔑 **Step 2: Installation and setup** 
```bash
git clone https://github.com/DefectDojo/django-DefectDojo
```
```bash
cd django-DefectDojo
```
```bash
./docker/docker-compose-check.sh
```
```bash
docker compose build
```

## 🔑 **Step 3: Start DefectDojo**
```bash
docker compose up -d
```

## 🔑 **Step 4: Obtain Admin Credentials**
```bash
docker compose logs -f initializer     
```
```bash
docker compose logs initializer | grep "Admin password:"
```
## 🔑 **Step 5: Access the DefectDojo Web UI**
```bash
http://your-ec2-public-ip:8080
```
- Username: admin
- Password: (Use the retrieved admin password from Step 4)

## 📦 **Step 6: Add a Product**  

1. Click on **Products** from the top navigation bar.  
2. Click **➕ Add Product**.  
3. Fill in the required details:  
- **Product Name**: (e.g., `WebApp Security Test`)  
- **Description**: (e.g., `Security testing for the internal web application`)  
- **Product Type**: Select a relevant type (e.g., `Web Application`)  
- **Business Criticality**: Choose (`High`, `Medium`, `Low`, etc.)  
- **Platform**: Specify if applicable (e.g., `AWS`, `On-Prem`, `Kubernetes`)  
- **Lifecycle**: Choose (`Production`, `Development`, etc.)  
4. Click **Save Product**.

## 🚀 **Step 7: Create an Engagement Under the Product**  


1. Navigate to **Products** and click on the **Product** you created in Step 2.  
2. Inside the product page, click **➕ Add Engagement**.  
3. Fill in the details:  
- **Product**: *(Automatically selected as the current product)*  
- **Engagement Name**: (e.g., `Q1 Security Assessment`)  
- **Lead**: Select the tester or security engineer leading the test.  
- **Target Start Date** / **Target End Date**: Set the timeframe.  
- **Testing Strategy**: Select (`CI/CD`, `Manual Testing`, etc.)  
- **Status**: Set to `In Progress`.  
4. Click **Save Engagement**.


## 🎯 **Next Steps: Import Security Scan Results through API**  

- **OWASP ZAP** (DAST)  
- **Trivy** (Container Security)  
- **SonarQube** (SAST)
---
        





