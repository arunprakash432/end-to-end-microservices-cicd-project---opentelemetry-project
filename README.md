<div align="center">

# 🔭 End-to-End Microservices CI/CD Pipeline
### OpenTelemetry Demo — Production-Grade Deployment on AWS EKS

[![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/eks/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI/CD-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)

*A fully instrumented, cloud-native microservices application demonstrating distributed tracing, metrics, and logs with a complete CI/CD pipeline from local Docker to production EKS.*

</div>

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
  - [Service Diagram](#service-diagram)
  - [Telemetry Data Flow](#telemetry-data-flow)
  - [Services & Languages](#services--languages)
- [Tech Stack](#-tech-stack)
- [Prerequisites & Installation](#️-prerequisites--installation)
- [Part 1 — Local Deployment with Docker Compose](#-part-1--local-deployment-with-docker-compose)
- [Part 2 — Production Deployment on AWS EKS](#️-part-2--production-deployment-on-aws-eks)
  - [Terraform — State Backend](#terraform--state-backend)
  - [Terraform — VPC & EKS Cluster](#terraform--vpc--eks-cluster)
  - [Kubernetes Deployment](#kubernetes-deployment)
  - [Exposing with LoadBalancer](#exposing-with-loadbalancer)
  - [ALB Ingress Controller Setup](#alb-ingress-controller-setup)
  - [Route 53 & Custom Domain](#route-53--custom-domain)
- [Part 3 — CI/CD Pipeline](#-part-3--cicd-pipeline)
  - [GitHub Actions](#github-actions)
  - [ArgoCD — GitOps Continuous Delivery](#argocd--gitops-continuous-delivery)
  - [Final Verification](#final-verification--application-live-on-custom-domain)
- [Project Screenshots Gallery](#-project-screenshots-gallery)

---

## 🌐 Project Overview

This project is an end-to-end implementation of the **[OpenTelemetry Demo Application](https://opentelemetry.io/docs/demo/)** — a real-world, polyglot microservices e-commerce platform — deployed on **AWS EKS** with a fully automated CI/CD pipeline.

The project demonstrates:

- **Distributed Observability** — Traces, metrics, and logs across 20+ microservices using OpenTelemetry
- **Infrastructure as Code** — All AWS resources provisioned with Terraform (remote state in S3 + DynamoDB locking)
- **Container Orchestration** — Kubernetes on Amazon EKS with VPC, IAM roles, node groups, and ALB Ingress
- **GitOps CI/CD** — GitHub Actions for CI (build & push images on PR merge) + ArgoCD for CD (self-healing deployments)
- **Production Networking** — Route 53 hosted zones mapped to a custom domain via ALB Ingress Controller

---

## 🏗 Architecture

### Service Diagram

> *The OpenTelemetry Demo is composed of 20+ microservices written in different programming languages, communicating over gRPC and HTTP, with a Locust-based load generator for simulated traffic.*

> **Reference:** [opentelemetry.io/docs/demo/architecture](https://opentelemetry.io/docs/demo/architecture/)

```
Internet / Load Generator / React Native App
        │
        ▼
  Frontend Proxy (Envoy)
        │
        ├──▶ Frontend (TypeScript)
        │         ├──▶ Ad Service (Java)
        │         ├──▶ Cart Service (.NET)  ──▶ Cache (Valkey)
        │         ├──▶ Currency Service (C++)
        │         ├──▶ Product Catalog (Go)
        │         ├──▶ Recommendation (Python)
        │         ├──▶ Checkout (Go)
        │         │       ├──▶ Payment (JavaScript)
        │         │       ├──▶ Email (Ruby)
        │         │       ├──▶ Shipping (Rust)  ──▶ Quote (PHP)
        │         │       └──▶ Kafka Queue
        │         │               ├──▶ Accounting (.NET)  ──▶ PostgreSQL
        │         │               └──▶ Fraud Detection (Kotlin)
        │         └──▶ Product Reviews (Python)  ──▶ LLM Service (Python)
        │
        ├──▶ Flagd UI (Elixir) — Feature flags management
        └──▶ Image Provider (nginx)

All services ──▶ Flagd (Go) — Feature flag evaluation
```

### Telemetry Data Flow

```
Microservices
    │
    ├──(OTLP gRPC)──▶ OTel Collector (:4317)
    └──(OTLP HTTP)──▶ OTel Collector (:4318)
                              │
              ┌───────────────┼────────────────┐
              ▼               ▼                ▼
         Prometheus      Jaeger           OpenSearch
         (metrics)       (traces)         (logs)
              │               │                │
              └───────────────┴────────────────┘
                              │
                           Grafana
                       (unified dashboards)
```

### Services & Languages

| Service | Language | Protocol | Description |
|---|---|---|---|
| Frontend Proxy | C++ (Envoy) | HTTP | Entry point for all external traffic |
| Frontend | TypeScript | gRPC/HTTP | Main web UI |
| Ad Service | Java | gRPC | Serves targeted advertisements |
| Cart Service | .NET | gRPC | Shopping cart with Valkey cache |
| Checkout | Go | gRPC | Orchestrates the order flow |
| Currency | C++ | gRPC | Currency conversion |
| Email | Ruby | HTTP | Order confirmation emails |
| Payment | JavaScript | gRPC | Payment processing |
| Product Catalog | Go | gRPC | Product listings |
| Product Reviews | Python | gRPC | Reviews + LLM integration |
| Recommendation | Python | gRPC | Product recommendations |
| Shipping | Rust | HTTP | Shipping cost estimation |
| Quote | PHP | HTTP | Shipping quotes |
| Accounting | .NET | TCP (Kafka) | Order accounting |
| Fraud Detection | Kotlin | TCP (Kafka) | Fraud analysis |
| LLM Service | Python | gRPC | AI-powered product reviews |
| Flagd | Go | gRPC | Feature flag service |
| Flagd UI | Elixir | HTTP | Feature flag management UI |
| Image Provider | nginx | HTTP | Product image hosting |
| Load Generator | Python (Locust) | HTTP | Simulated user traffic |

---

## 🛠 Tech Stack

| Category | Technology |
|---|---|
| **Cloud** | Amazon Web Services (AWS) |
| **Compute** | EC2 (t3.large), EKS (Managed Kubernetes) |
| **Networking** | VPC, ALB, Route 53 |
| **Storage** | S3 (Terraform state), DynamoDB (state lock) |
| **IaC** | Terraform (modules for VPC & EKS) |
| **Containers** | Docker, Docker Compose |
| **Orchestration** | Kubernetes (EKS), Helm |
| **CI** | GitHub Actions |
| **CD** | ArgoCD (GitOps) |
| **Ingress** | AWS ALB Ingress Controller |
| **DNS** | Route 53 + Hostinger (custom domain) |
| **Observability** | OpenTelemetry Collector, Jaeger, Prometheus, Grafana, OpenSearch |

---

## ⚙️ Prerequisites & Installation

### 1. AWS Account & IAM User

Create a dedicated IAM user with the required permissions:

```
IAM → Users → Create User → Attach Permissions
```

Required permissions: `AmazonEC2FullAccess`, `AmazonEKSFullAccess`, `IAMFullAccess`, `AmazonS3FullAccess`, `AmazonDynamoDBFullAccess`, `AmazonVPCFullAccess`

Then generate **Access Key** and **Secret Access Key** from Security Credentials.

![IAM User Created](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/01-iam-user.png)

![IAM User Details](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/02-iam-user.png)

![IAM User Access Key & Secret Key](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/03-iam-user-access-key.png)

### 2. Launch EC2 Instance

| Setting | Value |
|---|---|
| Name | `opentelemetry` |
| AMI | Ubuntu |
| Instance Type | `t3.large` |
| Key Pair | Assign existing or create new |

![EC2 Instance](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/04-ec2-instance.png)

Connect to the instance:

```bash
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
```

### 3. Install Required Tools

**Docker:**
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

**kubectl:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

**Terraform:**
```bash
sudo apt install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
```

**AWS CLI:**
```bash
sudo snap install aws-cli --classic
```

**Configure AWS credentials:**
```bash
aws configure
# Enter: Access Key ID, Secret Access Key, Region (ap-south-1), Output format (json)
```

---

## 🐳 Part 1 — Local Deployment with Docker Compose

Run the entire OpenTelemetry demo stack locally without Kubernetes.

### Clone the Repository

```bash
git clone <your-repo-url>
cd <repo-name>
```

### Start All Services

```bash
docker compose up -d
```

![Docker Compose Up](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/05-docker-compose.png)

![Docker PS — All Running Containers](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/06-docker-ps.png)

### Update EC2 Security Group

Open port `8080` in your EC2 security group — this is the Frontend Proxy port for the application UI.

```
EC2 → Security Groups → Inbound Rules → Add Rule
Type: Custom TCP | Port: 8080 | Source: 0.0.0.0/0
```

Access the application at: `http://<EC2-PUBLIC-IP>:8080`

![Browser Output — Application Running on Port 8080](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/07-browser-ouput.png)

### Tear Down

```bash
docker compose down --remove-orphans
```

---

## ☁️ Part 2 — Production Deployment on AWS EKS

### Terraform — State Backend

Before provisioning the main infrastructure, create a remote state backend to safely store the Terraform state file and prevent concurrent modifications.

```bash
cd backend/
terraform init
terraform plan
terraform apply
```

This provisions:
- **S3 Bucket** — Stores Terraform state file remotely
- **DynamoDB Table** — Provides state locking to prevent concurrent modifications

![S3 Bucket — Terraform State Locking](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/08-s3-bucket-state-locking.png)

![DynamoDB Table — State Locking](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/09-dynamo-table-state-locking.png)

---

### Terraform — VPC & EKS Cluster

Provision the VPC and EKS cluster using modular Terraform configurations:

```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

This creates:
- **VPC** with public and private subnets across availability zones
- **EKS Cluster** (`opentelemetry-project-eks-cluster`)
- **IAM Roles & Policies** for master node and worker nodes
- **Node Group** — managed EC2 worker nodes

![Terraform — VPC and EKS Cluster Provisioning](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/10-vpc-and-eks-cluster-terraform.png)

![EKS Cluster — AWS Console](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/11-eks-cluster.png)

![VPC — AWS Console](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/12-vpc.png)

---

### Kubernetes Deployment

**Update kubeconfig to connect to the EKS cluster:**

```bash
aws eks update-kubeconfig --region ap-south-1 --name opentelemetry-project-eks-cluster
```

**Verify the current context and cluster nodes:**

```bash
kubectl config current-context
kubectl get nodes
kubectl get all
```

**Create the Service Account:**

```bash
cd kubernetes/
kubectl apply -f serviceaccount.yaml
kubectl get sa
```

![Kubernetes Service Account Created](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/13-kubernetes-service-account.png)

**Deploy all microservices:**

```bash
kubectl apply -f completedeploy.yaml
```

![Kubernetes — Complete Deploy Applied](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/14-kubernetes-complete-deploy.png)

---

### Exposing with LoadBalancer

Check the frontend proxy service:

```bash
kubectl get svc | grep frontendproxy
```

Edit the service to change the type from `ClusterIP` to `LoadBalancer`:

```bash
kubectl edit svc opentelemetry-demo-frontendproxy
# Change: type: ClusterIP  →  type: LoadBalancer
```

![Change Service Type to LoadBalancer](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/15-change-to-loadbalancer.png)

![AWS LoadBalancer Created](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/16-aws-loadbalancer-created.png)

![Browser Output via LoadBalancer DNS](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/17-browser-output-using-loadbalancer.png)

> ⚠️ **Note:** Classic LoadBalancers have limitations — no path-based routing, higher cost per service, and no SSL termination at the ingress level. We replace this with an **ALB Ingress Controller** for production-grade traffic management.

---

### ALB Ingress Controller Setup

Revert the frontend proxy service type back to `NodePort` before proceeding:

```bash
kubectl edit svc opentelemetry-demo-frontendproxy
# Change: type: LoadBalancer  →  type: NodePort
```

#### Step 1 — Setup IAM OIDC Provider

The OIDC provider allows Kubernetes service accounts to assume IAM roles, which is required for the ALB controller to make AWS API calls.

```bash
export cluster_name=opentelemetry-project-eks-cluster
```

```bash
oidc_id=$(aws eks describe-cluster --name $cluster_name \
  --query "cluster.identity.oidc.issuer" \
  --output text | cut -d '/' -f 5)
```

Check if an IAM OIDC provider is already configured:

```bash
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

If not configured, associate it:

```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

![IAM OIDC Provider Configured](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/18-iam-oidc-provider.png)

#### Step 2 — Create IAM Policy for ALB Controller

Download the official IAM policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

Create the IAM policy in AWS:

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

![IAM Policy — AWSLoadBalancerControllerIAMPolicy](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/19-iam-policy.png)

#### Step 3 — Create IAM Role & Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

![eksctl IAM Service Account Created](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/20-eksctl-iam-service-account.png)

#### Step 4 — Install ALB Controller via Helm

Add the EKS Helm chart repository and update:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

![Helm Installation](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/21-helm-installation.png)

Install the AWS Load Balancer Controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```

Verify the controller deployment is running:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

![AWS Load Balancer Controller Installed](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/22-aws-loadbalancer-controller.png)

#### Step 5 — Verify IAM Policy Permissions

> ⚠️ **Important:** If you cannot see the LoadBalancer address when running `kubectl get ing`, the `AWSLoadBalancerControllerIAMPolicy` may be missing the `elasticloadbalancing:DescribeListenerAttributes` permission.

Check if the permission exists:

```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy \
      --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
      --query 'Policy.DefaultVersionId' --output text)
```

![Check IAM Policy Version](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/23-check-iam-policy-version.png)

If the permission is missing, download the current policy, add the missing statement, then create a new default policy version:

```bash
# Download current policy
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy \
      --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
      --query 'Policy.DefaultVersionId' --output text) \
    --query 'PolicyVersion.Document' --output json > policy.json
```

Add the following block inside the `Statement` array of `policy.json`:

```json
{
  "Effect": "Allow",
  "Action": "elasticloadbalancing:DescribeListenerAttributes",
  "Resource": "*"
}
```

Apply the updated policy as the new default version:

```bash
aws iam create-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://policy.json \
    --set-as-default
```

![AWS Load Balancer Controller — Verified Running](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/24-aws-loadbalancer-controller.png)

![AWS Load Balancer Controller — Full Details](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/25-details-of-aws-loadbalancer-controller.png)

#### Step 6 — Deploy the Ingress Resource

```bash
cd kubernetes/frontendproxy/
kubectl apply -f ingress.yaml
```

**Testing with local DNS (Windows) before Route 53 propagation:**

To test the ingress locally before DNS is fully propagated, update the Windows `hosts` file to map your domain to the ALB IP.

![Windows DNS Hosts File — Location](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/26-path-of-dns-windows.png)

Verify the Ingress and ALB DNS are working:

```bash
kubectl apply -f ingress.yaml
nslookup <your-alb-dns>
```

![Apply Ingress and nslookup Output](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/27-apply-ingress-and-nslookup.png)

![Local DNS Configuration Change](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/28-change-the-local-dns-configuration.png)

![Browser Output via Ingress — Homepage](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/29-browser-output.png)

![Browser Output via Ingress — Full App](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/30-browser-output.png)

---

### Route 53 & Custom Domain

Map a custom domain to the ALB using Route 53 for production DNS resolution.

#### Step 1 — Create a Hosted Zone

```
Route 53 → Hosted Zones → Create Hosted Zone → Enter your domain name
```

![Route 53 Hosted Zones](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/31-route-53-hosted-zone.png)

![Create Hosted Zone](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/32-create-hosted-zone.png)

#### Step 2 — Create DNS Records

Create an **A Record** (Alias) pointing to your ALB:

```
Record Type : A — Alias
Alias Target: Your ALB DNS name
```

![Create DNS Records — Step 1](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/33-create-records.png)

![Create DNS Records — Step 2](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/34-create-records-2.png)

#### Step 3 — Update Nameservers at Domain Registrar

Copy the 4 nameservers from the Route 53 hosted zone:

![Route 53 Nameserver Details](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/35-route53-nameserver-details.png)

Paste them into your domain registrar (Hostinger) to delegate DNS to Route 53:

![Hostinger — Nameserver Update](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/36-hostinger-nameserver-change.png)

#### Step 4 — Update Hostname in ingress.yaml

Update the `host` field in `kubernetes/frontendproxy/ingress.yaml` to match your custom domain, then re-apply:

```bash
kubectl apply -f ingress.yaml
```

![Update Hostname in ingress.yaml](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/37-change-the-hostname-in-ingress-yaml-file.png)

#### Step 5 — Verify LoadBalancer Logs

Confirm traffic is being routed correctly through the ALB:

![LoadBalancer Access Logs](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/38-loadbalancer-logs.png)

---

## 🚀 Part 3 — CI/CD Pipeline

### GitHub Actions

The CI pipeline is triggered automatically on **pull requests** and **merges to main**. When code changes are pushed, GitHub Actions builds a new Docker image for the affected service and pushes it to DockerHub.

**Workflow:**
1. Developer pushes code changes and opens a Pull Request
2. GitHub Actions CI workflow triggers on the PR
3. Docker image is built for the changed service
4. Image is tagged and pushed to DockerHub
5. ArgoCD detects the updated image/manifest and auto-syncs to the EKS cluster

![GitHub Actions — Build Triggered](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/39-github-actions-build.png)

![GitHub Actions — Build Steps in Progress](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/40-github-actions-build-2.png)

![GitHub Actions — Pipeline Completed Successfully](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/41-github-actions-3.png)

![DockerHub — Image Pushed Successfully](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/42-dockerhub-image-upload.png)

---

### ArgoCD — GitOps Continuous Delivery

ArgoCD continuously reconciles the desired state (GitHub repository) with the live state (Kubernetes cluster), providing **self-healing**, **auto-sync**, and **drift detection**.

**Install Helm:**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Install ArgoCD via Helm:**
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd --create-namespace
```

**Retrieve the initial admin password:**
```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

![ArgoCD — Retrieve Login Password](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/43-arogcd-login-password.png)

**Get the ArgoCD Server External IP:**
```bash
kubectl get svc argocd-server -n argocd
```

![ArgoCD — Server External IP via LoadBalancer](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/44-argocd-svc-externalip.png)

![ArgoCD LoadBalancer and Application LoadBalancer — AWS Console](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/45-argocd-loadbalancer-and-application-loadbalancer.png)

**Access the ArgoCD UI:**

Navigate to the External IP in your browser using `https://`. Or use port-forward for local access:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Access at: https://localhost:8080  (note: HTTPS, not HTTP)
```

**Configure the Application in ArgoCD UI:**

1. Login with username `admin` and the retrieved password
2. Click **New App** and fill in:
   - **Application Name:** `opentelemetry-demo`
   - **Repository URL:** Your GitHub repository URL
   - **Path:** `kubernetes/`
   - **Cluster URL:** `https://kubernetes.default.svc` (in-cluster)
   - **Namespace:** `default`
   - **Sync Policy:** Enable **Auto Sync** + **Self Heal** + **Prune**
3. Click **Create** then **Sync**

![ArgoCD — Configure Project Step 1](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/46-argocd-project-1.png)

![ArgoCD — Configure Project Step 2](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/47-argocd-project-2.png)

![ArgoCD — Configure Project Step 3](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/48-argocd-project-3.png)

![ArgoCD — Application Deployed & Healthy](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/49-argocd-deployement.png)

![ArgoCD — Microservice Dependency Graph](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/50-argocd-graph.png)

> **Result:** Any change pushed to the `kubernetes/` directory in your repository is automatically detected and deployed to your EKS cluster within minutes — no manual `kubectl apply` required.

---

### Final Verification — Application Live on Custom Domain

With Route 53 DNS propagated and ArgoCD synced, the full application is live on your custom domain:

![Browser Output — Custom Domain Live (1)](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/51-browser-output-domain-name-1.png)

![Browser Output — Custom Domain Live (2)](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/52-browser-output-2.png)

---

## 📸 Project Screenshots Gallery

| # | Stage | Screenshot |
|---|---|---|
| 01 | IAM User Created | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/01-iam-user.png) |
| 02 | IAM User Details | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/02-iam-user.png) |
| 03 | IAM Access Key & Secret Key | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/03-iam-user-access-key.png) |
| 04 | EC2 Instance | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/04-ec2-instance.png) |
| 05 | Docker Compose Up | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/05-docker-compose.png) |
| 06 | Docker PS | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/06-docker-ps.png) |
| 07 | Browser Output (Port 8080) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/07-browser-ouput.png) |
| 08 | S3 Bucket — State Locking | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/08-s3-bucket-state-locking.png) |
| 09 | DynamoDB — State Locking | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/09-dynamo-table-state-locking.png) |
| 10 | Terraform VPC & EKS | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/10-vpc-and-eks-cluster-terraform.png) |
| 11 | EKS Cluster | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/11-eks-cluster.png) |
| 12 | VPC | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/12-vpc.png) |
| 13 | Kubernetes Service Account | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/13-kubernetes-service-account.png) |
| 14 | Kubernetes Complete Deploy | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/14-kubernetes-complete-deploy.png) |
| 15 | Change to LoadBalancer | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/15-change-to-loadbalancer.png) |
| 16 | AWS LoadBalancer Created | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/16-aws-loadbalancer-created.png) |
| 17 | Browser via LoadBalancer | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/17-browser-output-using-loadbalancer.png) |
| 18 | IAM OIDC Provider | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/18-iam-oidc-provider.png) |
| 19 | IAM Policy | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/19-iam-policy.png) |
| 20 | eksctl IAM Service Account | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/20-eksctl-iam-service-account.png) |
| 21 | Helm Installation | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/21-helm-installation.png) |
| 22 | AWS Load Balancer Controller | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/22-aws-loadbalancer-controller.png) |
| 23 | Check IAM Policy Version | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/23-check-iam-policy-version.png) |
| 24 | AWS Load Balancer Controller (2) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/24-aws-loadbalancer-controller.png) |
| 25 | Load Balancer Controller Details | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/25-details-of-aws-loadbalancer-controller.png) |
| 26 | Windows DNS Hosts File Path | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/26-path-of-dns-windows.png) |
| 27 | Apply Ingress & nslookup | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/27-apply-ingress-and-nslookup.png) |
| 28 | Local DNS Configuration | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/28-change-the-local-dns-configuration.png) |
| 29 | Browser Output via Ingress (1) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/29-browser-output.png) |
| 30 | Browser Output via Ingress (2) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/30-browser-output.png) |
| 31 | Route 53 Hosted Zone | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/31-route-53-hosted-zone.png) |
| 32 | Create Hosted Zone | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/32-create-hosted-zone.png) |
| 33 | Create Records (1) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/33-create-records.png) |
| 34 | Create Records (2) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/34-create-records-2.png) |
| 35 | Route 53 Nameserver Details | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/35-route53-nameserver-details.png) |
| 36 | Hostinger Nameserver Change | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/36-hostinger-nameserver-change.png) |
| 37 | Update Hostname in ingress.yaml | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/37-change-the-hostname-in-ingress-yaml-file.png) |
| 38 | LoadBalancer Logs | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/38-loadbalancer-logs.png) |
| 39 | GitHub Actions Build (1) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/39-github-actions-build.png) |
| 40 | GitHub Actions Build (2) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/40-github-actions-build-2.png) |
| 41 | GitHub Actions Pipeline (3) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/41-github-actions-3.png) |
| 42 | DockerHub Image Upload | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/42-dockerhub-image-upload.png) |
| 43 | ArgoCD Login Password | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/43-arogcd-login-password.png) |
| 44 | ArgoCD Service External IP | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/44-argocd-svc-externalip.png) |
| 45 | ArgoCD + App LoadBalancer | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/45-argocd-loadbalancer-and-application-loadbalancer.png) |
| 46 | ArgoCD Project Config (1) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/46-argocd-project-1.png) |
| 47 | ArgoCD Project Config (2) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/47-argocd-project-2.png) |
| 48 | ArgoCD Project Config (3) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/48-argocd-project-3.png) |
| 49 | ArgoCD — App Deployed & Healthy | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/49-argocd-deployement.png) |
| 50 | ArgoCD — Service Graph | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/50-argocd-graph.png) |
| 51 | Live on Custom Domain (1) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/51-browser-output-domain-name-1.png) |
| 52 | Live on Custom Domain (2) | ![](https://raw.githubusercontent.com/arunprakash432/end-to-end-microservices-cicd-project---opentelemetry-project/main/assets/screenshots/52-browser-output-2.png) |

---

<div align="center">

**Built with ❤️ — OpenTelemetry | AWS | Terraform | Kubernetes | GitHub Actions | ArgoCD**

*Reference: [OpenTelemetry Demo Docs](https://opentelemetry.io/docs/demo/) · [Architecture](https://opentelemetry.io/docs/demo/architecture/)*

</div>
