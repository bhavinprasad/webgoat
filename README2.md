# WebGoat CI/CD Pipeline

This repository contains the Jenkins pipeline to automate the Continuous Integration and Continuous Deployment (CI/CD) process for the **WebGoat** project. The pipeline integrates multiple security scans, builds, Docker container management, and deployment tasks. The goal is to ensure high-quality code and deploy the application after performing static and dynamic analysis.

## Pipeline Overview

The pipeline is divided into several stages:

1. **Clone** the repository.
2. **SAST (Static Application Security Testing)** using **SonarQube**.
3. **Quality Gate** to check whether the code passes the defined quality standards.
4. **OWASP Dependency Check** to detect known vulnerabilities in project dependencies.
5. **Trivy File System Scan** for vulnerabilities in the project file system.
6. **Build and Deploy** to Docker.
7. **Trivy Docker Image Scan** to check for vulnerabilities in the Docker image.
8. **Deploy to DockerHub** for storage of the latest image.
9. **Dynamic Analysis** using **OWASP ZAP** for runtime security testing.
10. **Import SonarQube Results to DefectDojo** for tracking issues.

---

## Setup

### Prerequisites

Before using this pipeline, make sure you have the following setup:

1. **Jenkins Server** with the necessary plugins:
   - Docker plugin
   - SonarQube Scanner
   - OWASP Dependency-Check plugin
   - Trivy plugin
   - DefectDojo plugin
   - SSH Credentials plugin
   - Git plugin

2. **External Tools & Services**:
   - **SonarQube** server for Static Application Security Testing.
   - **Docker Hub** for container registry.
   - **Trivy** for scanning Docker images and file systems.
   - **OWASP ZAP** for dynamic security testing.
   - **DefectDojo** server for vulnerability tracking and reporting.
   - **Prometheus** for gathering and storing metrics data.
   - **Grafana** for creating dashboards and visualizing metrics from Prometheus.
   - **Dojo Reporting** integrated with DefectDojo for detailed security reports.

---

## Separate Setup Guides

For each tool and service used in the CI/CD pipeline, there is a dedicated README file with detailed instructions on how to set it up. Below is the list of tools and their respective setup guides:

- **SonarQube Setup**: For setting up **SonarQube**, refer to the [SONARQUBE.md](SONARQUBE.md).
- **Trivy Setup**: For setting up **Trivy**, refer to the [TRIVY.md](TRIVY.md).
- **OWASP ZAP Setup**: For setting up **OWASP ZAP**, refer to the [ZAP.md](ZAP.md).
- **DefectDojo Setup**: For setting up **DefectDojo**, refer to the [DEFECTDOJO.md](DEFECTDOJO.md).
- **Prometheus Setup**: For setting up **Prometheus**, refer to the [PROMETHEUS.md](PROMETHEUS.md).
- **Grafana Setup**: For setting up **Grafana**, refer to the [GRAFANA.md](GRAFANA.md).


}
