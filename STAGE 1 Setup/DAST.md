## 🚀 Setting Up a DAST Machine (OWASP ZAP)



### 🔑 **Step 1: Connect to the DAST Machine via SSH**  

Use the following command to SSH into the machine:  

```bash
ssh -i /path/to/your-key.pem ubuntu@your-dast-machine-ip
```

### 🛠 Step 2: Install Docker

Update the package list and install Docker:

```bash
sudo apt update && sudo apt install -y docker.io
```

Enable and start the Docker service:

```bash
sudo systemctl enable --now docker
```

Verify the installation:

```bash
docker --version
```

### 🔥 Step 3: Run OWASP ZAP in Docker

Run OWASP ZAP Full Scan using Docker:

```bash
sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://your-webgoat-url.com
```

**Explanation of parameters:**

- `--rm` → Removes the container after execution
- `-v /home/ubuntu:/zap/wrk/:rw` → Mounts the home directory to `/zap/wrk/` with read/write permissions
- `-t` → Runs the container in a terminal session
- `owasp/zap2docker-stable` → Uses the latest stable ZAP Docker image
- `zap-full-scan.py -t http://your-webgoat-url.com` → Starts a full scan on the target URL

**Note:** Replace `your-webgoat-url.com` with the actual target website to scan.

### 📂 Step 4: Access Scan Results

Since we mounted `/home/ubuntu` to `/zap/wrk/`, the scan results will be stored in:

```bash
/home/ubuntu/
```

To list the results:

```bash
ls -lh /home/ubuntu

