# 🚀 Complete CI/CD Pipeline Setup  
**Tools Used:** Jenkins | SonarQube | Docker | Kubernetes (Minikube) | ArgoCD | GitHub  

---

## 🖥️ 1. Create and Configure EC2 Instance
1. Launch an **Ubuntu EC2 instance**  
   - Type: `t2.large` (2 CPU, 8 GB RAM)  
2. Connect to the instance using **MobaXterm** or SSH.  
3. Clone or download your project repository.  
   ```bash
   git clone <your-repo-url>
   cd <repo-folder>
   ```

---

## ☕ 2. Install Java
```bash
sudo apt update
sudo apt install openjdk-17-jre -y
java -version
```

---

## ⚙️ 3. Install and Configure Jenkins
```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
ps -ef | grep jenkins
```

### 🔓 Jenkins Setup
1. Allow port **8080** in your EC2 security group.  
2. Retrieve the initial admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
3. Open Jenkins in browser:  
   `http://<EC2-PUBLIC-IP>:8080`
4. Unlock Jenkins → Continue as admin → Skip plugin install → Add pipeline plugin.

### 🔗 Integrate Jenkins with GitHub
- In Jenkins, go to **Manage Jenkins → Credentials** → Add GitHub credentials.  
- Configure repository URL in your pipeline job.

---

## 🐳 4. Install Docker and Configure for Jenkins
```bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
systemctl restart docker
```

Restart Jenkins after this:
```bash
http://<ec2-public-ip>:8080/restart
```

---

## 🔍 5. Install and Configure SonarQube
```bash
sudo apt update && sudo apt install unzip -y
sudo adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip sonarqube-10.4.1.88267.zip
chmod -R 755 sonarqube-10.4.1.88267
chown -R sonarqube:sonarqube sonarqube-10.4.1.88267
cd sonarqube-10.4.1.88267/bin/linux-x86-64/
./sonar.sh start
```

### 🔐 Connect Jenkins with SonarQube
1. Access SonarQube in browser:  
   `http://<EC2-IP>:9000`
2. Login (default: admin/admin), change password.
3. Generate a **SonarQube Token**.
4. Add token to Jenkins under **Manage Jenkins → Credentials → Secret Text**.
5. Install **SonarQube Scanner plugin** in Jenkins.
6. Configure SonarQube server in Jenkins:
   ```
   Manage Jenkins → Configure System → SonarQube Servers
   ```

---

## 🧰 6. Docker Credentials in Jenkins
- Go to **Manage Jenkins → Credentials → Add Credentials**
  - **Kind:** Username and Password  
  - **ID:** `docker-cred`

---

## ☁️ 7. Create Kubernetes Cluster (Minikube)
```bash
minikube start --driver=docker
# OR
minikube start --driver=virtualbox --no-vtx-check
```

### Install Cert-Manager
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl get pods --namespace cert-manager
```

---

## 🧩 8. Install ArgoCD on Kubernetes

### Install Operator Lifecycle Manager (OLM)
```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.34.0/install.sh | bash -s v0.34.0
```

### Install ArgoCD Operator
```bash
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
kubectl get csv -n operators
```

### Create Basic ArgoCD Instance
```bash
vim argocd-basic.yml
```
Paste:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```

Apply:
```bash
kubectl apply -f argocd-basic.yml
kubectl get pods
```

### Expose ArgoCD Service
```bash
kubectl edit svc example-argocd-server
# Change type from ClusterIP → NodePort
minikube service example-argocd-server
minikube service list
```

### Access ArgoCD Dashboard
- Get the admin password:
  ```bash
  kubectl get secret
  kubectl edit secret example-argocd-cluster
  echo <base64-password> | base64 -d
  ```
- Username: `admin`  
- Password: `<decoded-password>`

---

## 🧪 9. Configure Jenkins Pipeline
- Install plugins:
  - Docker Pipeline
  - SonarQube Scanner
  - Kubernetes CLI
  - Slack Notification (optional)
- Create a **Pipeline Job** in Jenkins.
- Add your GitHub repo URL.
- Define `Jenkinsfile` with build, sonar scan, docker push, and deploy steps.

---

## 🧾 10. Connect ArgoCD with Your Application
In ArgoCD UI → Click **New App**  
Fill details:
| Field | Value |
|-------|--------|
| Application Name | `test` |
| Project Name | `default` |
| Sync Policy | `Automatic` |
| Repository URL | `<your-github-repo>` |
| Path | `<manifest-folder-path>` |
| Cluster URL | `https://kubernetes.default.svc` |
| Namespace | `default` |

Click **Create**.

---

## 🔁 11. Verify Full CI/CD Flow
1. Commit a code change → Jenkins triggers build.  
2. SonarQube analyzes code quality.  
3. Docker image builds & pushes to Docker Hub.  
4. Deployment manifest is updated with new image tag.  
5. ArgoCD automatically detects changes and syncs to the Kubernetes cluster.  
6. Verify the new image is deployed:
   ```bash
   kubectl get pods
   kubectl get svc
   ```

---

## 🧩 12. Optional Checks
- **SonarQube:** Review code vulnerabilities.  
- **Docker Hub:** Confirm image pushed with new tag.  
- **ArgoCD:** View Sync & History.  
- **Kubernetes:** Verify pod updates.  

---

## ✅ CI/CD Pipeline Summary
| Stage | Tool | Description |
|--------|------|-------------|
| Build | Jenkins | Runs Maven/Gradle build |
| Code Quality | SonarQube | Analyzes vulnerabilities |
| Image Build | Docker | Creates & tags app image |
| Push | Docker Hub | Stores container image |
| Deploy | ArgoCD | Syncs with Kubernetes |
| Monitor | Slack (optional) | Sends notifications |

---

**🎯 Final Result:**  
A fully automated CI/CD pipeline that builds, tests, analyzes, containerizes, and deploys your application into a Kubernetes cluster using ArgoCD.
