# Ultimate DevOps Project Demo

## Introduction
This document outlines the steps to set up and run the Ultimate DevOps Project Demo. It includes instructions for creating IAM users, configuring EC2 instances, installing necessary tools, running the project locally, deploying microservices, and utilizing Kubernetes, Terraform, and GitHub Actions.

---

## AWS IAM and EC2 Setup

### Create an IAM User
1. Navigate to the IAM Dashboard in AWS.
2. Create a new user:
   - **Name**: `DevOps-user`
   - **Authentication**: Enable
   - **Policy**: Attach `AdministratorAccess` policy.
   
### Create an EC2 Instance
1. **Instance Type**: `t2.large`
2. **Storage**: EBS with 15 GB.
3. Switch to the IAM user `DevOps-user` to perform the following steps.

---

## Install Tools on EC2 Instance
1. Install Docker:
   ```bash
   sudo apt update
   sudo apt install docker.io
   ```
2. Install Kubectl:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   ```
3. Install Terraform:
   ```bash
   sudo apt update
   sudo apt install -y gnupg software-properties-common curl
   curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
   sudo apt update
   sudo apt install terraform
   ```

---

## Run Project Locally

### Clone the Repository
```bash
git clone https://github.com/lavkeshp/ultimate-devops-project-demo.git
```

### Run Docker Compose
```bash
docker-compose up
```

#### Troubleshoot: No Storage Left on Device
1. Increase volume size:
   - Navigate to EC2 > Volumes > Actions > Modify.
   - Set the size to 30 GB.
2. Resize the volume:
   ```bash
   $ df -h
   $ lsblk
   $ sudo apt install cloud-guest-utils
   $ sudo growpart /dev/xvda
   $ sudo resize2fs /dev/xvda1
   $ df -h
   ```

---

## Microservice Setup

### Product Catalog
1. Delete the existing Dockerfile.
2. Create a new Dockerfile and debug issues if necessary.
3. Build and run the image:
   ```bash
   docker build -t lavkeshp/products:v1 .
   docker run -d lavkeshp/products:v1
   ```

### Ad Service
1. Run microservice:
   ```bash
   ./gradlew installDist
   export AD_PORT=9099
   src/ad/build/install/oteldemo/bin/Ad
   ```
2. Create a Dockerfile, build the image, and run the container.

### Recommendation Service
1. Write a Dockerfile.
2. Build and run the container.

---

## Docker Compose
Use Docker Compose to manage multiple microservices.

---

## Kubernetes (K8s) Setup

### Prerequisites
1. Install AWS CLI:
   ```bash
   sudo apt install awscli
   aws configure
   ```
2. Install Terraform and configure AWS credentials:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

### Create EKS Cluster
1. Update kubeconfig:
   ```bash
   aws eks update-kubeconfig --region us-east-2 --name my-eks-cluster
   ```
2. Verify the cluster:
   ```bash
   kubectl config view
   kubectl config current-context
   kubectl get nodes
   ```

### ALB Setup
1. Associate IAM OIDC Provider:
   ```bash
   export cluster_name=my-eks-cluster
   oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
   eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
   ```
2. Create IAM Policy:
   ```bash
   curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
   aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
   ```
3. Install Helm:
   ```bash
   curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
   sudo apt-get install apt-transport-https --yes
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
   sudo apt-get update
   sudo apt-get install helm
   ```
4. Install ALB using Helm:
   ```bash
   helm repo add eks https://aws.github.io/eks-charts
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$cluster_name --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-2 --set vpcId=vpc-067b283f76b950dee
   ```

---

## GitHub Actions

### Workflow
1. Create a workflow file in `.github/workflows/ci.yaml`.
2. Define jobs for build, code quality, Docker, and Kubernetes.

### GitOps with ArgoCD
1. Install ArgoCD:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
2. Edit ArgoCD Service:
   ```bash
   kubectl edit svc argocd-server -n argocd
   ```
   - Change type to `LoadBalancer`.
3. Log in to ArgoCD:
   ```bash
   kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
   ```
   - Default credentials: `admin` and decoded password.
4. Create a project and provide the repository URL and path to Kubernetes manifests.

---

## Additional Notes
- Buy a domain name and configure it with Route 53.
- Edit the IP and domain name in the Ingress configuration.