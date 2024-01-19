
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
