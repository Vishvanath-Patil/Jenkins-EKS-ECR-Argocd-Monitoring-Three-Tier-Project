# Table of Contents

1. [Phase 1: Jenkins Setup](#phase-1-jenkins-setup)
    - [Step 1: Launch EC2 (Ubuntu 22.04)](#step-1-launch-ec2-ubuntu-2204)
    - [Step 2: CI/CD Setup](#step-2-cicd-setup)
        - [Install Jenkins for Automation](#install-jenkins-for-automation)
        - [Install Docker for Sonarqube installation and Container service](#install-docker-for-sonarqube-installation-and-container-service)
    - [Phase 2: Security](#phase-2-security)
        - [Install SonarQube and Trivy](#install-sonarqube-and-trivy)
        - [Integrate SonarQube and Configure](#integrate-sonarqube-and-configure)
        - [Install Necessary Plugins in Jenkins](#install-necessary-plugins-in-jenkins)
        - [Configure Java and Nodejs in Global Tool Configuration](#configure-java-and-nodejs-in-global-tool-configuration)
        - [SonarQube](#sonarqube)
        - [Install Dependency-Check and Docker Tools in Jenkins](#install-dependency-check-and-docker-tools-in-jenkins)
        - [Install Dependency-Check Plugin](#install-dependency-check-plugin)
        - [Configure Dependency-Check Tool](#configure-dependency-check-tool)
        - [Install Docker Tools and Docker Plugins](#install-docker-tools-and-docker-plugins)
        - [Add ECR credentials for your AWS credentials](#add-ecr-credentials-for-your-aws-credentials)
        - [Add DockerHub Credentials](#add-dockerhub-credentials)
        - [Configure CI/CD Pipeline in Jenkins for frontend](#configure-cicd-pipeline-in-jenkins-for-frontend)
2. [Phase 3: Baston Host, EKS Cluster & ECR Setup](#phase-3-baston-host-eks-cluster--ecr-setup)
    - [Prerequisites](#prerequisites)
    - [Steps](#steps)
        - [Step 1: EC2 Setup](#step-1-ec2-setup)
        - [Step 2: Clone the Code](#step-2-clone-the-code)
        - [Step 3: IAM Configuration](#step-3-iam-configuration)
        - [Step 4: Install AWS CLI v2](#step-4-install-aws-cli-v2)
        - [Step 5: Install Docker](#step-5-install-docker)
        - [Step 6: Install kubectl](#step-6-install-kubectl)
        - [Step 7: Install eksctl](#step-7-install-eksctl)
        - [Step 8: Setup EKS Cluster](#step-8-setup-eks-cluster)
        - [Step 9: Create ECR Repository](#step-9-create-ecr-repository)
        - [Step 10: Run Manifests](#step-10-run-manifests)
        - [Step 11: Install AWS Load Balancer](#step-11-install-aws-load-balancer)
        - [Step 12: Deploy AWS Load Balancer Controller](#step-12-deploy-aws-load-balancer-controller)
        - [Cleanup](#cleanup)
    - [Contribution Guidelines](#contribution-guidelines)
    - [Support](#support)



# Deploy Three-Tier-Application Using Jenkins, EKS (ClouFormation), ECR, ArgoCD and Monitoring Stetup (Prometheus, Grafana, Loki and Cloudwatch) - Project!

# Phase 1: Jenkins Setup

## Step 1: Launch EC2 (Ubuntu 22.04):

- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.


## Step 2: CI/CD Setup

1. **Install Jenkins for Automation:**
    - Install Jenkins on the EC2 instance to automate deployment:
    Install Java
    
    ```bash
    sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version
    openjdk version "17.0.8" 2023-07-18
    OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
    OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
    
    #jenkins
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    ```
  ## Verify Java Installation
        ```bash
        java --version
        ```
  ## Verify Jenkins Installation
        ```bash
        sudo systemctl status jenkins
        ```
  ## Should show like this 
  ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/5164e24a-fe3a-4feb-a76c-bc516c19f4bb)

    - Access Jenkins in a web browser using the public IP of your EC2 instance.
        
        publicIp:8080
  ## Jenkins Server Console look like this 
  ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/cbe15aec-d50e-491c-933b-73166c2d2d8b)

    - Setup Jenkins User Profile
### Run Command to get Administration Password
 ```
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword      
 ```
### Output would be like this 
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/fc14809a-6380-4c06-b0ae-206e2bc2459b)

- Install Suggested plugins
  ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/775dea30-03d0-40c7-882b-6b3548a8bcd6)
- Create Profile
- ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/dd0357fe-a005-41d0-9139-8764342a3520)
- ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/2f81be89-dd84-4c97-841a-2aa9ee2159dc)

  


## Step 3: Install Docker for Sonarqube installation and Container service:

- Set up Docker on the EC2 instance:
    
    ```bash
    
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    sudo su                            # To avoid docker login failed error
    sudo usermod -aG docker jenkins
    sudo systemctl restart jenkins
    ```
### Verify Docker Intallation

 ```
 docker ps
 docker images
 sudo systemctl status docker
 ```
### Output should show like this
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/6f0012a5-f173-4f86-a9a5-d9bda679c210)
 

  
# Phase 2: Security

1. ## Install SonarQube and Trivy:
    - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
        
        sonarqube
        ```
        docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
        ```
        ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/7934b786-3ec1-442c-aa40-787630fadf0d)

        
        To access: 
        
        publicIP:9000 (by default username & password is admin)
        ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/e0e0fea5-1fe1-4378-b13e-fe69926dcaf9)
        ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/f95fb295-fec1-45c0-a3fa-f6643edc694a)

        
        To install Trivy:
        ```
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy        
        ```
        
        to scan image using trivy
        ```
        trivy image <imageid>
        ```
3. ## Integrate SonarQube and Configure:
    - Integrate SonarQube with your CI/CD pipeline.
    - Configure SonarQube to analyze code for quality and security issues.

2. ## Install Necessary Plugins in Jenkins:

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 Eclipse Temurin Installer (Install without restart)

2 SonarQube Scanner (Install without restart)

3 NodeJs Plugin (Install Without restart)

4 Email Extension Plugin

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/e45e9502-ab04-4502-8956-3237c61f815f)


### **Configure Java and Nodejs in Global Tool Configuration**

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/0f16f77f-6973-4bd0-984b-4c9f543dc108)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/ccca2ed1-763f-4189-9056-479b98310dd0)



### SonarQube

Create the token

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/d6bdb429-0a11-4900-aa4a-60364f6b15cd)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/e350852b-c3e3-4c8e-bfa5-08259e59bceb)



Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/e9f2deb4-3f9c-435f-9cd8-99fc1ad601ab)

After adding sonar token

Click on Apply and Save
## Sonar Server Configuration on Jenkins 
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/db57b948-f775-4b2f-949e-b1f55bb418e9)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/dd749629-2f51-4a51-bc12-c8a2e05a6ef8)



**The Configure System option** is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/a2bbfcfa-244c-4601-ba3b-c1b4b084f2fd)

Create *todo* Project on Sonarqube for **Sonarqube Analysis** Stage Integration 
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/3cdacca9-763f-44be-b470-29219e5f920a)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/178d1175-30bc-4882-9b3e-487af7f5e7bf)

Certainly, here are the instructions without step numbers:

## Install Dependency-Check and Docker Tools in Jenkins

## Install Dependency-Check Plugin:

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "OWASP Dependency-Check."
- Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.
  ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/42d806ff-bf0d-4655-9f2c-be6bb4bc51d6)


## Configure Dependency-Check Tool:

- After installing the Dependency-Check plugin, you need to configure the tool.
- Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."
- Find the section for "OWASP Dependency-Check."
- Add the tool's name, e.g., "DP-Check."
- Save your settings.
  ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/77205dc0-22f5-4de0-95b4-868a7c907254)


## Install Docker Tools and Docker Plugins:

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "Docker."
- Check the following Docker-related plugins:
  - Docker
  - Docker Commons
  - Docker Pipeline
  - Docker API
  - docker-build-step
  - Amazon ECR
  - Amazon EKS
- Click on the "Install without restart" button to install these plugins.
  ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/ad3d7df0-173b-4979-8536-a7f3cb0efb36)

  ## Configure Docker Tool:

- After installing the Dependency-Check plugin, you need to configure the tool.
- Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."
- Find the section for "Docker"
- Add the tool's name, e.g., "docker"
- Save your settings.
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/f61e6891-b57c-4eb4-a214-94757f07be92)

Create Security Credrentials for jenkins 
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/77bf1061-df0b-4803-bc16-fa6f846a58b0)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/f6a94ec9-9ba6-4024-994c-e92c9a751217)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/3a3940a1-fb37-419a-b2fe-db065efa9553)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/080b1644-2fbc-4e50-9f66-10917dc304a8)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/971356e1-6841-4849-adb3-14a2197170ec)






## Add ECR credentials for your AWS credentials using the following steps:
- Open the Jenkins web interface.

- Navigate to "Manage Jenkins" > "Manage Credentials."

- In the Credentials page, click on "(global)" or a specific domain where you want to store your AWS credentials.

- Click on "Add Credentials" on the left side.

- Choose the kind of credentials to add. In this case, select "Secret text" for AWS Access Key ID and Secret Access Key.

Fill in the necessary information:

- Secret: Your AWS Access Key ID or Secret Access Key.
  ID: Provide a unique ID for these credentials. This ID will be used in your Jenkins pipeline script.
  Description: Optionally, provide a description for better identification.
  Click on "OK" to save the credentials.

- Now, you can use the ID you provided in the credentialsId field in your Jenkins pipeline script. For example, if you set the ID to aws-ecr-credentials, your pipeline

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/00ae68fb-a03e-46ac-a2f8-5342472d107c)


## Add DockerHub Credentials:

- To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
  - Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
  - Click on "System" and then "Global credentials (unrestricted)."
  - Click on "Add Credentials" on the left side.
  - Choose "Secret text" as the kind of credentials.
  - Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
  - Click "OK" to save your DockerHub credentials.
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/e0afc152-4ae9-4093-bb46-6813493d09fe)


Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.

1. ## Configure CI/CD Pipeline in Jenkins for frontend:
- Create a CI/CD pipeline in Jenkins to automate your application deployment.
- ![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/4455cc09-f959-48fb-b6ba-54ed9d4ee381)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/9814b926-3bf7-43c1-a081-6b363b53a29f)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/2daabe0e-69ce-4450-b8d8-14f4fc4d3cea)


```groovy
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node14'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Todo \
                    -Dsonar.projectKey=Todo '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t frontend frontend"
                       sh "docker tag frontend vishwa3877/frontend:${BUILD_NUMBER}"
                       sh "docker push vishwa3877/frontend:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project"
            GIT_USER_NAME = "Vishvanath-Patil"
        }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "pvishva93@gmail.com"
                    git config user.name "Vishvanath Patil"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" k8s_manifests/frontend-deployment.yaml
                    git add k8s_manifests/frontend-deployment.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }    
} 
}
        stage("TRIVY"){
            steps{
                sh "trivy image vishwa3877/frontend:${BUILD_NUMBER} > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name frontend -p 8081:80 vishwa3877/frontend:${BUILD_NUMBER}'
            }
        }
    }
}
```
![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/8aeaae0d-7474-4ead-a68a-d74a5e70f155)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/74f5855c-1171-4d5d-8ca7-37a94c8631d9)

![image](https://github.com/Vishvanath-Patil/Jenkins-EKS-ECR-Argocd-Monitoring-Three-Tier-Project/assets/130968991/a8075e0c-b0fc-450c-ac6a-2a6758815f3c)


# Phase 3: Baston Host, EKS Cluster & ECR Setup

## Prerequisites
- Basic knowledge of Docker, and AWS services.
- An AWS account with necessary permissions.

## Steps 

### Step 1: EC2 Setup
- Launch an Ubuntu (t2.micro) instance in your favourite region (eg. region `us-west-2`).
- SSH into the instance from your local machine.

### Step 2: Clone the Code:
- Update all the packages and then clone the code.
- Clone your application's code repository onto the EC2 instance:

``` shell
git clone https://github.com/Vishvanath-Patil/Three-Tier-Application-Deployment-Project.git
```

### Step 3: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.
  
### Step 4: Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Step 5: Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
sudo usermod -aG docker $USER
sudo systemctl status docker
#If the Docker daemon is not running, start it using
sudo systemctl start docker
#Check Docker Socket Permissions:
#Ensure that the permissions on the Docker socket file (/var/run/docker.sock) allow the user or the docker group to access it. You can check the permissions with:
ls -l /var/run/docker.sock
#It should have read and write permissions for the user or be owned by the docker group.
sudo systemctl restart docker #Restart docker

```

### Step 6: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 7: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 8: Setup EKS Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```
### Note :
make sure replace by your region and clustername

### Step 9: Create ECR Repository
### Prerequisites to Push docker image to ECR Repo is `aws configure` this has done in Step.4
#### Step 9.1 : Search ECR and Click on Get Started
![image](https://github.com/Vishvanath-Patil/Three-Tier-Application-Deployment-Project/assets/130968991/1225fd9d-f3ab-40b5-89e7-25c4c6ce6e6e)
#### Step 9.2 : Choose Repository type `Public` and fill Repository Name `worokshop/three-tier-backend` then Click on Create Repository
![image](https://github.com/Vishvanath-Patil/Three-Tier-Application-Deployment-Project/assets/130968991/efcaa18a-2cd3-453e-baf0-149a9a4e15dc)
#### Step 9.3 : Then Open `worokshop/three-tier-backend` and click on View Push Commands 
![image](https://github.com/Vishvanath-Patil/Three-Tier-Application-Deployment-Project/assets/130968991/986d5d17-ca41-4066-95a0-4486a41db705)
#### Step 9.4 : Click on View Push Commands
![image](https://github.com/Vishvanath-Patil/Three-Tier-Application-Deployment-Project/assets/130968991/0cc09b8f-ed20-4368-8828-55798189d718)
#### Step 9.5 : Use following Commands to Build Docker Image and Push to ECR Repository 
![image](https://github.com/Vishvanath-Patil/Three-Tier-Application-Deployment-Project/assets/130968991/914ff518-3433-479f-bc92-3bf48da6ce43)

###Note :- should RUN these commands inside `Dockerfile` directory 

### Step 10: Run Manifests
``` shell
kubectl create namespace workshop
kubectl config set-context --current --namespace workshop
kubectl apply -f .
kubectl delete -f .
```

### Step 11: Install AWS Load Balancer
``` shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

### Note : 
Replace AWS Account ID (626072240565), AmazonEKSLoadBalancerControllerRole, AWSLoadBalancerControllerIAMPolicy, and region 

### Step 12: Deploy AWS Load Balancer Controller
``` shell
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```

### Step 13: Configure AWS Load Balancer Controller DNS with GoDaddy Domain Name
- Step 1: Obtain the Public DNS of AWS Load Balancer
  Navigate to the AWS Management Console.
  Open the Amazon EC2 console and choose "Load Balancers" from the navigation pane.
  Select your load balancer and note down its Public DNS.
- Step 2: Access GoDaddy Account
  Log in to your GoDaddy account.
- Step 3: Access DNS Management
  In your GoDaddy account, go to the "DNS Management" or "Domain Management" section.
- Step 4: Add a CNAME Record
  Add a new CNAME record to map your domain to the AWS Load Balancer.

  - Record Type: CNAME
  - Host: Enter the subdomain or leave it blank for the root domain.
  - Points to: Enter the Public DNS of your AWS Load Balancer.
  - TTL (Time to Live): Set the desired TTL value.

     #### Example:
     - Host: www
     - Points to: Your AWS Load Balancer Public DNS
Note: This step associates your domain (or subdomain) with the AWS Load Balancer.

- Step 5: Save Changes
  Save the changes made to the DNS records.
- Step 6: Wait for DNS Propagation
  DNS changes may take some time to propagate. Wait for the changes to take effect.
- Step 7: Test the Configuration
  Open a web browser and navigate to your domain or subdomain (e.g., http://www.yourdomain.com).

Note: It may take a while for the DNS changes to propagate completely.

#### Additional Considerations
Ensure that the security groups associated with your AWS Load Balancer allow traffic on the necessary ports.
Verify the SSL/TLS configuration if your application uses HTTPS.
Monitor AWS Route 53 or GoDaddy DNS settings for any issues or errors.
By following these steps, you'll configure your GoDaddy domain to point to the AWS Load Balancer Controller. Adjust the instructions based on your specific domain configuration and requirements.

# Phase 4: ArgoCD Setup

# Phase 5: Monitoring SetUp

### Cleanup
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name three-tier-cluster --region us-west-2
```

## Contribution Guidelines
- Fork the repository and create your feature branch.
- Deploy the application, adding your creative enhancements.
- Ensure your code adheres to the project's style and contribution guidelines.
- Submit a Pull Request with a detailed description of your changes.


## Support
For any queries or issues, please open an issue in the repository.

---
Happy Learning! 🚀👨‍💻👩‍💻
