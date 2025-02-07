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
        





