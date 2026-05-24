# Three-Tier Application on Amazon EKS
 
A production-ready, cloud-native three-tier web application deployed on Amazon Elastic Kubernetes Service (EKS). This project demonstrates modern DevOps practices including containerization, orchestration, and infrastructure as code.

## 🎯 Project Overview
 
This is a full-stack web application deployed on AWS EKS (Elastic Kubernetes Service) with three distinct layers:
 
1. **Frontend Layer** - React-based single-page application
2. **Backend Layer** - Node.js/Express REST API
3. **Database Layer** - MongoDB for data persistence

### Key Features
 
✅ **Containerized Application** - Docker images for all application tiers  
✅ **Kubernetes Native** - Complete K8s manifests for deployment  
✅ **Auto-Scaling Ready** - Configured for horizontal pod autoscaling  
✅ **Health Checks** - Liveness and readiness probes for reliability  
✅ **Load Balancing** - AWS ALB integration for traffic distribution  
✅ **Data Persistence** - Persistent volumes for database storage  
✅ **Secrets Management** - Kubernetes secrets for credentials  
✅ **Rolling Updates** - Zero-downtime deployments  
 
---

## 📦 Prerequisites
 
### Local Machine
 
- **Git** - Version control
- **AWS Account** - With appropriate IAM permissions
- **Docker** - For building container images (optional, images provided)
- **kubectl** - Kubernetes command-line tool (v1.19+)
- **eksctl** - AWS EKS cluster management tool
- **AWS CLI v2** - AWS command-line interface
### AWS Resources
 
- IAM user with `AdministratorAccess` policy
- AWS Access Key and Secret Access Key
- EC2 key pair for SSH access
- Sufficient service quotas for EKS, EC2, and load balancers
---

### IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.

### EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region `us-west-2`).
- SSH into the instance from your local machine.

### Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Setup EKS Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

### Run Manifests
``` shell
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```

### Install AWS Load Balancer
``` shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

### Deploy AWS Load Balancer Controller
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
- To clean up rest of the stuff and not incure any cost
```
Stop or Terminate the EC2 instance created in step 2.
Delete the Load Balancer created in step 9 and 10.
Go to EC2 console, access security group section and delete security groups created in previous steps
```

