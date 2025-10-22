	FULL STEPS OF CICD PIPELINE
> Create ec2 instance (ubuntu, t2.large (2cpu, 8gb))
> Run locally README.MD
> Configure ec2 on mobaxterm
>Install Java
    sudo apt update
    sudo apt install openjdk-17-jre
    Verify Java is Installed
java -version
	Now, you can proceed with installing Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins
ps -ef | grep jenkins
	Configure traffic port
	Cat password and paste on Jenkins
	Skip and continue as admin
	Select pipeline 
	Intigerate jenkins repo in git with jenkins
	Apply and Save
	Install docker pipeline plugin
	Install sonarqube scanner plugin
	Configure a Sonar Server
sudo apt update && sudo apt install unzip -y

sudo adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-10.4.1.88267
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267
cd sonarqube-10.4.1.88267/bin/linux-x86-64/
./sonar.sh start
	To Anthuanticate Jenkins with Sonarqube
	Generate token
	Add credential
	Docker Slave Configuration
sudo apt update
sudo apt install docker.io
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
Once you are done with the above steps, it is better to restart Jenkins.
http://<ec2-instance-public-ip>:8080/restart

	Create Kubernetes Cluster
minikube start --driver=virtualbox --no-vtx-check
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl get pods --namespace cert-manager
Install on Kubernetes
Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.34.0/install.sh | bash -s v0.34.0
Install the operator by running the following command
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster.
After install, watch your operator come up using next command.
kubectl get csv -n operators
To use it, checkout the custom resource definitions (CRDs) introduced by this operator to start using it.



Add Credentials of Docker name docker-cred
   Docker Username and Password
Add Credentials of Github {ID github / secret: token}
Restart Jenkins


	Check Jenkins pipeline works or not
	Refresh sonar 
	Check sonar for code vulnarablites
	Check docker image on docker hub which is push by Jenkins pipeline
	Go to docs in https://operatorhub.io/operator/argocd-operator
	Go to usage basics
	Copy the example
	vim argocd-basic.yml

apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}

	paste in “vim argocd-basic.yml” file
	kubectl apply -f argocd-basic.yml
	kubectl get pods (wait for getting in ready state)
	kubectl get svc
	change example-agrocd-server ClusterIP to NodePort
	kubectl edit svc example-agrocd-server
	minikube service argocd-server
	minikube service list
	check pods are running
	kubectl get pods
	paste url to browser
	kubectl get secret
	kubectl edit secret example-argocd-cluster 
	echo <psswd> | base64 -d
	username : admin password : paste it
	click new app
	Application Name : test
Project Name: default
SYCN POLICY: Automatic
Repository url : repo
Path : manifest path
Cluster URL: https://kubernetes.default.svc
Namespace: default
	Click create
	Try change image build number in deployment “kubectl edit deploy spring-boot-app”
	Check Diff
	Then sycn












  



