# Jenkins on AWS EC2 — CI/CD Mini Lab

## Overview

This project demonstrates how to deploy a **self-hosted Jenkins CI server on an Ubuntu AWS EC2 instance using Docker** and create a multi-stage Jenkins pipeline.

The lab was designed as a practical introduction to running Jenkins in a cloud environment and understanding the basic components of a CI pipeline, including builds, testing, parallel execution, and artifact archiving.

---

## Architecture

```text
                    AWS Cloud
                       │
                       ▼
              ┌─────────────────┐
              │   Ubuntu EC2    │
              │                 │
              │  Docker Engine  │
              │       │         │
              │       ▼         │
              │  Jenkins LTS    │
              │   Container     │
              │                 │
              │  Port 8080      │
              │  Port 50000     │
              └────────┬────────┘
                       │
                       ▼
                Jenkins Pipeline
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    Initialize       Build           Test
                                     /   \
                                    ▼     ▼
                              Unit Test   Lint
                                     │
                                     ▼
                                  Artifact
```

---

# 1. AWS EC2 Setup

An Ubuntu EC2 instance was deployed to host the Jenkins environment.

Because Jenkins and Docker can consume a reasonable amount of memory, especially during builds, a **4 GB swap file** was configured on the instance.

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

To make the swap persistent across reboots:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

The memory configuration was verified using:

```bash
free -h
```

The lab environment showed approximately 908 MiB of RAM and 4 GiB of configured swap.

---

# 2. Install Docker

Docker and Docker Compose were installed on the Ubuntu EC2 instance.

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
```

The Ubuntu user was added to the Docker group:

```bash
sudo usermod -aG docker ubuntu
newgrp docker
```

Docker installation was verified with:

```bash
docker --version
docker compose version
```

The lab environment used:

```text
Docker version 29.1.3
Docker Compose version 2.40.3
```

---

# 3. Create Jenkins Lab Directory

A dedicated directory was created for the Jenkins lab:

```bash
mkdir ~/jenkins-lab
cd ~/jenkins-lab
```

---

# 4. Deploy Jenkins Using Docker

Jenkins was deployed as a Docker container using the Jenkins LTS image with JDK 17.

```bash
docker run -d \
  --name jenkins-lab \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --restart=on-failure \
  jenkins/jenkins:lts-jdk17
```

### Configuration

| Configuration       | Value                       |
| ------------------- | --------------------------- |
| Container name      | `jenkins-lab`               |
| Jenkins image       | `jenkins/jenkins:lts-jdk17` |
| Jenkins Web UI      | Port `8080`                 |
| Agent communication | Port `50000`                |
| Persistent volume   | `jenkins_home`              |
| Restart policy      | `on-failure`                |

The container was verified with:

```bash
docker ps
```

Jenkins was running successfully with the configured ports exposed.

---

# 5. Jenkins Initial Setup

After starting the Jenkins container, the initial administrator password was retrieved from the Jenkins container:

```bash
docker exec jenkins-lab \
cat /var/jenkins_home/secrets/initialAdminPassword
```

The password was then used to unlock Jenkins through the web interface.

The Jenkins initial setup was completed by:

* Unlocking Jenkins
* Installing the required plugins
* Creating an administrator account
* Accessing the Jenkins dashboard

> **Security Note:** Never commit the Jenkins initial administrator password, credentials, API tokens, SSH keys, or other secrets to GitHub.

---

# 6. Create the Jenkins Pipeline

A multi-stage pipeline was created to simulate a basic CI workflow for an application named **MyLabApp**.

The pipeline contained the following stages:

```text
Initialize
    │
    ▼
Build
    │
    ▼
Test
 ┌──┴──┐
 ▼     ▼
Unit   Lint
Test
 └──┬──┘
    ▼
Artifact
```

---

# 7. Initialize Stage

The pipeline begins by displaying a message indicating that the build has started.

```text
Starting build for MyLabApp...
```

This stage provides a simple initialization point before the build process begins.

---

# 8. Build Stage

The Build stage simulated application compilation.

```bash
echo "Compiling code..."
sleep 2
```

After the simulated compilation completed, Jenkins reported:

```text
Build successful.
```

This demonstrates how shell commands can be executed from a Jenkins pipeline.

---

# 9. Test Stage

The testing stage used Jenkins **parallel execution**.

Two branches were executed simultaneously:

### Unit Test

```bash
echo "Running unit tests..."
sleep 1
```

### Lint

```bash
echo "Running code linter..."
sleep 1
```

The pipeline therefore executed:

```text
             Test
              │
        ┌─────┴─────┐
        │           │
    Unit Test      Lint
        │           │
        └─────┬─────┘
              │
           Continue
```

Parallel execution can reduce pipeline execution time when independent tasks do not need to wait for each other.

---

# 10. Artifact Stage

After the test stage completed, the pipeline packaged the application.

A sample artifact was created using:

```bash
touch app_v1.tar.gz
```

The artifact was then archived by Jenkins using the `archiveArtifacts` pipeline step.

The Jenkins console showed:

```text
Packaging application...
Archiving artifacts
Recording fingerprints
```

The resulting artifact was:

```text
app_v1.tar.gz
```

---

# 11. Pipeline Result

The complete pipeline executed successfully.

The Jenkins console output showed:

```text
[Pipeline] End of Pipeline
Finished: SUCCESS
```

The pipeline therefore completed the complete flow:

```text
Initialize
     ↓
Build
     ↓
Test
 ┌───┴────┐
 ↓        ↓
Unit     Lint
 Test
 └───┬────┘
     ↓
 Artifact
     ↓
  SUCCESS
```

---

# 12. What I Learned

This lab provided hands-on experience with:

* AWS EC2
* Ubuntu Linux
* Linux swap configuration
* Docker
* Docker volumes
* Docker container management
* Jenkins installation
* Jenkins administration
* Jenkins plugins
* Jenkins pipelines
* Pipeline stages
* Shell execution from Jenkins
* Parallel pipeline execution
* Artifact creation
* Artifact archiving
* Jenkins build logs

More importantly, the lab helped connect the concepts of **cloud infrastructure, containers, and CI automation** in a single environment.

---

# 13. Practical Use Case

A setup like this can be useful for:

* Small development teams
* Internal CI environments
* Proof-of-concept projects
* Learning and experimentation
* Temporary automation environments
* Self-hosted CI/CD infrastructure

Instead of using a managed CI platform, Jenkins can be hosted on an EC2 instance and customized according to the project's requirements.

---

# 14. Future Improvements

This lab currently uses simulated build and testing commands. A more realistic CI/CD implementation could extend the pipeline with:

* GitHub source-code integration
* Automated Git checkout
* Real application compilation
* Real unit tests
* Code quality analysis
* Docker image building
* Container image scanning
* Dependency vulnerability scanning
* Secrets scanning
* Docker image publishing
* Deployment to AWS
* Infrastructure-as-Code integration
* Notifications for failed builds

This would move the lab closer to a complete **DevSecOps CI/CD pipeline**.

---

## Final Result

Successfully deployed Jenkins on an **Ubuntu AWS EC2 instance using Docker** and executed a multi-stage CI pipeline containing:

**Initialize → Build → Parallel Test → Artifact**

The pipeline completed successfully and archived the generated application artifact.

This project provided a practical foundation for understanding how **Jenkins, Docker, Linux, and AWS EC2** can be combined to build a self-hosted CI environment.

---

## Technologies Used

```text
AWS EC2
Ubuntu Linux
Docker
Docker Compose
Jenkins LTS
JDK 17
Shell/Bash
Jenkins Pipeline
```

## Status

**Completed ✅**
