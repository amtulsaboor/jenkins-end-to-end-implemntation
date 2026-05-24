# End-to-End CI/CD Pipeline Implementation using Jenkins, SonarQube, Docker, Kubernetes & ArgoCD
<img width="1693" height="929" alt="a2662f17-432b-4c09-bb1e-bd32b691b99c" src="https://github.com/user-attachments/assets/9ca99a78-8a85-4c4d-9933-e5ecab5415be" />



## Project Overview

In this project, we will implement a complete End-to-End CI/CD pipeline for a Java application using:

* Jenkins
* SonarQube
* Docker
* Kubernetes
* Argo CD
* Amazon Web Services EC2
* GitHub
* Docker Hub

This project demonstrates a real-world DevOps workflow where:

1. Developers push code to GitHub.
2. Jenkins pulls the code.
3. Maven builds the application.
4. SonarQube performs code quality analysis.
5. Docker builds and pushes the image to DockerHub.
6. Kubernetes deploys the application.
7. ArgoCD automates Continuous Deployment using GitOps.

---

# Architecture Flow

```text
Developer → GitHub Repository → Jenkins Pipeline
                    ↓
             SonarQube Analysis
                    ↓
              Docker Build
                    ↓
            Push to DockerHub
                    ↓
            Kubernetes Cluster
                    ↓
                 ArgoCD
                    ↓
          Automatic Deployment
```
<img width="1014" height="1280" alt="PHOTO-2026-05-23-18-38-23" src="https://github.com/user-attachments/assets/d07dec8d-7c74-4271-aa28-6c58cba9c19e" />

---

# Prerequisites

Before starting the project make sure you have:

* AWS Account
* GitHub Account
* DockerHub Account
* Basic knowledge of Linux commands
* Minikube installed on local machine
* kubectl installed on local machine
* Docker Desktop installed and running

---

# Step 1: Create EC2 Instance

Login to [AWS Console](https://aws.amazon.com/console/?utm_source=chatgpt.com) and create an Ubuntu EC2 instance with:

| Configuration  | Value             |
| -------------- | ----------------- |
| Instance Type  | t2.xlarge         |
| Storage        | 30 GB             |
| OS             | Ubuntu            |
| Security Group | Allow All Traffic |

> Important: Delete the instance after project completion to avoid AWS billing charges.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 4 51 20 PM" src="https://github.com/user-attachments/assets/a8514199-a23c-4720-b256-ebca1e0041ee" />

---

# Step 2: Connect to EC2 Instance

SSH into the instance:

```bash
ssh -i your-key.pem ubuntu@<PUBLIC-IP>
```
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 4 53 11 PM" src="https://github.com/user-attachments/assets/97573699-c360-4cab-b360-fae5f861987c" />

---

# Step 3: Install Java 21

Java is a prerequisite for Jenkins.

Update packages:

```bash
sudo apt update
```

Install Java 21:

```bash
sudo apt install openjdk-21-jdk -y
```

Verify Java installation:

```bash
java -version
```

Expected output should display Java 21.

---

# Step 4: Install Jenkins

Add Jenkins repository key:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Add Jenkins repository:

```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

Update packages:

```bash
sudo apt update
```

Install Jenkins:

```bash
sudo apt install jenkins -y
```
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 4 56 15 PM" src="https://github.com/user-attachments/assets/bfdc4ff5-a745-4db3-9223-0f98e4308806" />

Start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check Jenkins status:

```bash
sudo systemctl status jenkins
```

---

# Step 5: Access Jenkins

Jenkins runs on port `8080`.

Open browser:

```text
http://<PUBLIC-IP>:8080
```

Retrieve Jenkins admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it in Jenkins setup page.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 4 59 10 PM" src="https://github.com/user-attachments/assets/05860c7b-6ecd-4df9-b6cd-d94e7657dc3d" />

Install suggested plugins and complete setup.

---

# Step 6: Configure Jenkins Pipeline

Inside Jenkins:

```text
New Item → Pipeline → OK
```

Under Pipeline section:

```text
Definition → Pipeline script from SCM
```

Choose:

```text
SCM → Git
```

Paste your GitHub repository URL and provide the Jenkinsfile path.

Save the pipeline.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 5 01 55 PM" src="https://github.com/user-attachments/assets/8965e106-8a17-45d1-bdca-d56658ed6597" />

---

# Step 7: Install Required Jenkins Plugins

Go to:

```text
Manage Jenkins → Plugins
```

Install:

* Docker Pipeline Plugin
* SonarQube Scanner Plugin

Restart Jenkins after installation.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 5 03 12 PM" src="https://github.com/user-attachments/assets/900a78e7-e874-4fe0-8a37-b2cf29dc476b" />

---

# Step 8: Install SonarQube

Switch to root user:

```bash
sudo su
```

Create SonarQube user:

```bash
adduser sonarqube
```

Install unzip:

```bash
apt install unzip -y
```

---

# Step 9: Install Java 17 for SonarQube

> Important: SonarQube works properly with Java 17. It may crash on Java 21 or Java 25.

Install Java 17:

```bash
apt install openjdk-17-jdk -y
```

Verify version:

```bash
java -version
```

---

# Step 10: Download and Configure SonarQube

Switch to SonarQube user:

```bash
su - sonarqube
```

Download SonarQube:

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
```

Unzip package:

```bash
unzip sonarqube-10.4.1.88267.zip
```
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 5 04 50 PM" src="https://github.com/user-attachments/assets/05072a20-c7a3-4c15-b45b-a083cde4b9e1" />
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 5 05 48 PM" src="https://github.com/user-attachments/assets/81701825-8111-41d6-bda8-d5a7581988bb" />

Navigate to SonarQube directory:

```bash
cd sonarqube-10.4.1.88267/bin/linux-x86-64/
```

Start SonarQube:

```bash
./sonar.sh start
```

Check SonarQube status:

```bash
./sonar.sh status
```
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 5 17 40 PM" src="https://github.com/user-attachments/assets/cec4a47f-254e-4ab7-8425-dff59bcf05f2" />

---

# Step 11: Access SonarQube

SonarQube runs on port `9000`.

Open browser:

```text
http://<PUBLIC-IP>:9000
```

Default credentials:

| Username | Password |
| -------- | -------- |
| admin    | admin    |

Change the password after first login.

---

# Step 12: Generate SonarQube Token

Go to:

```text
My Account → Security → Generate Tokens
```

Provide token name and generate token.

Copy the generated token.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 11 43 09 PM" src="https://github.com/user-attachments/assets/1ed633bd-22bc-4b00-8f9f-2916be4e55fa" />

---

# Step 13: Add SonarQube Token in Jenkins

Navigate to:

```text
Manage Jenkins → Credentials → System → Global Credentials
```

Add Credentials:

| Field  | Value           |
| ------ | --------------- |
| Kind   | Secret Text     |
| Secret | SonarQube Token |
| ID     | sonarqube       |

Save credentials.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 11 44 40 PM" src="https://github.com/user-attachments/assets/abb3a130-f229-4906-94e6-22e1bdd222bc" />

---

# Step 14: Install Docker

Switch to root user:

```bash
sudo su
```

Install Docker:

```bash
apt install docker.io -y
```

Enable Docker:

```bash
systemctl enable docker
systemctl start docker
```

Check Docker status:

```bash
systemctl status docker
```

---

# Step 15: Grant Docker Permissions

Grant Docker access to Jenkins and Ubuntu users:

```bash
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

---

# Step 16: Install Maven

Install Maven:

```bash
sudo apt install maven -y
```

Verify Maven installation:

```bash
mvn -version
```

---

# Step 17: Configure DockerHub Credentials

Inside Jenkins:

```text
Manage Jenkins → Credentials → Global → Add Credentials
```

Choose:

| Field    | Value                  |
| -------- | ---------------------- |
| Kind     | Username with Password |
| Username | DockerHub Username     |
| Password | DockerHub Password     |

Save credentials.
<img width="1710" height="1107" alt="Screenshot 2026-05-23 at 11 59 26 PM" src="https://github.com/user-attachments/assets/e1d97708-30e9-4923-aa15-9f45752650ce" />

---

# Step 18: Configure GitHub Token

Generate GitHub Personal Access Token from:

[GitHub Tokens](https://github.com/settings/tokens?utm_source=chatgpt.com)

Add it inside Jenkins:

```text
Manage Jenkins → Credentials → Global → Add Credentials
```

Choose:

| Field  | Value        |
| ------ | ------------ |
| Kind   | Secret Text  |
| Secret | GitHub Token |

Save credentials.
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 12 02 15 AM" src="https://github.com/user-attachments/assets/17ebff3a-2d04-4134-83e3-4df702b54c7a" />

---

# Step 19: Run CI Pipeline

Go to Jenkins project:

```text
Build Now
```

Pipeline stages will execute:

* Git Clone
* Maven Build
* SonarQube Scan
* Docker Build
* Docker Push

If build fails initially:

* Check Console Output
* Troubleshoot errors carefully
* Verify credentials
* Verify Docker permissions
* Verify SonarQube token

Once successful:

* SonarQube dashboard will display analysis results.
* DockerHub will contain Docker image with tags.
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 12 59 42 AM" src="https://github.com/user-attachments/assets/0d5d1539-d954-4cb2-8273-72120546b2de" />
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 1 06 05 AM" src="https://github.com/user-attachments/assets/fb670844-a926-4e0d-afab-b3cec576a05a" />
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 1 05 13 AM" src="https://github.com/user-attachments/assets/74bb03af-00cf-4832-adea-43ab1f63d6f9" />

---

# Step 20: Install Minikube on Local Machine

Make sure:

* Docker Desktop is running
* kubectl is installed
* Minikube is installed

Start Minikube:

```bash
minikube start --driver=docker --cpus=2 --memory=4096 --disk-size=20g --kubernetes-version=v1.28.3
```

Verify cluster:

```bash
kubectl get nodes
```

---

# Step 21: Install ArgoCD Operator

Visit:

[OperatorHub.io](https://operatorhub.io/?utm_source=chatgpt.com)

Install OLM:

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.42.0/install.sh | bash -s v0.42.0
```

Install ArgoCD Operator:

```bash
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```

Verify installation:

```bash
kubectl get csv -n operators
```

---

# Step 22: Configure ArgoCD in Detail

After installing the ArgoCD Operator, the next step is to create an ArgoCD Custom Resource file and deploy it inside the Kubernetes cluster.

---

# Step 22.1: Create Namespace (Optional but Recommended)

Create a dedicated namespace for ArgoCD:

```bash
kubectl create namespace argocd
```

Verify namespace:

```bash
kubectl get ns
```

---

# Step 22.2: Create ArgoCD YAML File

Create the YAML file:

```bash
vim argocd-basic.yml
```

Press `i` to enter insert mode and paste the following configuration.

---

# Complete ArgoCD YAML Configuration

```yaml
apiVersion: argoproj.io/v1beta1
kind: ArgoCD
metadata:
  name: example-argocd
  namespace: operators
spec:
  server:
    service:
      type: NodePort
```
save it.
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 1 28 08 AM" src="https://github.com/user-attachments/assets/4ccdc8bd-993c-4e42-8f42-ee873d144a81" />

---

# Step 22.3: Apply the YAML File

Run:

```bash
kubectl apply -f argocd-basic.yml
```

Expected output:

```text
argocd.argoproj.io/example-argocd created
```

---

# Step 22.4: Verify ArgoCD Pods

Check pods:

```bash
kubectl get pods -n operators
```

You will see ArgoCD pods running.

Example:

```text
NAME                                                 READY   STATUS
example-argocd-application-controller-0              1/1     Running
example-argocd-redis-xxxxx                           1/1     Running
example-argocd-repo-server-xxxxx                     1/1     Running
example-argocd-server-xxxxx                          1/1     Running
```

---

# Step 22.5: Verify Services

Run:

```bash
kubectl get svc -n operators
```

Expected output:

```text
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
example-argocd-server     NodePort   10.xx.xx.xx     <none>        443:32000/TCP
```

Important part:

```text
443:32000/TCP
```

Here:

* `443` = Internal service port
* `32000` = NodePort exposed externally

---

# Step 22.7: Get Minikube IP

Run:

```bash
minikube ip
```

Example output:

```text
192.168.49.2
```
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 1 29 14 AM" src="https://github.com/user-attachments/assets/acf7483e-0ae6-46d1-9dfa-123c7414e7c6" />

---

# Step 22.8: Access ArgoCD Dashboard

Open browser:

```text
https://<MINIKUBE-IP>:<NODEPORT>
```

Example:

```text
https://192.168.49.2:32000
```

If browser shows warning:

```text
Your connection is not private
```

Click:

```text
Advanced → Proceed
```

because ArgoCD uses self-signed certificates.
<img width="1710" height="1107" alt="Screenshot 2026-05-24 at 1 41 34 AM" src="https://github.com/user-attachments/assets/3b4b17c0-e5d9-4f30-ba90-93862a07fc53" />

---

# Step 22.9: Get ArgoCD Admin Password

Run:

```bash
kubectl get secret -n operators
```

You will see:

```text
example-argocd-cluster
example-argocd-secret
```

Now retrieve password:

```bash
kubectl get secret example-argocd-cluster -n operators -o yaml
```

OR use this easier command:

```bash
kubectl get secret example-argocd-cluster -n operators -o jsonpath="{.data.admin\.password}" | base64 -d
```

This will display the admin password.

---

# Step 22.10: Login to ArgoCD

Use:

| Field    | Value              |
| -------- | ------------------ |
| Username | admin              |
| Password | Retrieved Password |

Login successfully into ArgoCD Dashboard.

---

# Step 23: Deploy Application using ArgoCD

Now we will connect GitHub repository with ArgoCD for GitOps deployment.

---

# Step 23.1: Create Application YAML File

Create file:

```bash
vim app.yml
```

Paste the following YAML.

---

# Complete Application YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: springboot-demo
  namespace: operators

spec:
  project: default

  source:
    repoURL: https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY.git
    targetRevision: HEAD
    path: kubernetes

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
|

---

# Step 23.2: Save the File

In vim:

```bash
ESC
:wq
```

---

# Step 23.3: Apply Application YAML

Run:

```bash
kubectl apply -f app.yml
```

Expected output:

```text
application.argoproj.io/springboot-demo created
```

---

# Step 23.4: Verify Application

Run:

```bash
kubectl get applications -n operators
```

Expected output:

```text
NAME               SYNC STATUS   HEALTH STATUS
springboot-demo    Synced        Healthy
```

---

# Step 23.5: Verify Pods

Run:

```bash
kubectl get pods
```

You will see application pods running.

---

# Step 23.6: Verify Services

Run:

```bash
kubectl get svc
```

Example:

```text
NAME              TYPE       CLUSTER-IP      PORT(S)
springboot-svc    NodePort   10.xx.xx.xx     80:30080/TCP
```

---

# Step 23.7: Access Application

Open browser:

```text
http://<MINIKUBE-IP>:30080
```

Your Java application will be deployed successfully.

---

# Step 24: GitOps Workflow

Now complete GitOps workflow is active.

Whenever code changes are pushed to:

[GitHub](https://github.com/?utm_source=chatgpt.com)

The following happens automatically:

```text
GitHub → Jenkins → SonarQube → DockerHub → Kubernetes → ArgoCD
```

ArgoCD continuously monitors the repository and automatically updates Kubernetes deployment.

---

# Important Commands Summary

## Kubernetes Commands

```bash
kubectl get pods
kubectl get svc
kubectl get deployments
kubectl get applications -n operators
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

# Delete Minikube Cluster

After completion:

```bash
minikube delete
```

---

# Delete EC2 Instance

Go to:

[AWS EC2 Console](https://console.aws.amazon.com/ec2/?utm_source=chatgpt.com)

Terminate the instance to avoid billing charges.
