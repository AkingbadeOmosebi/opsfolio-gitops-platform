# Opsfolio: GitOps Platform

## Automated Deployment Pipeline | ArgoCD | AWS EKS | Terraform Infrastructure

[![Opsfolio](https://img.shields.io/badge/Opsfolio-GitOps_Platform-2563eb?style=flat-square)](https://github.com/AkingbadeOmosebi/opsfolio-gitops-platform)
[![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/eks/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=flat-square&logo=terraform)](https://www.terraform.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=flat-square&logo=argo)](https://argoproj.github.io/cd/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?style=flat-square&logo=kubernetes)](https://kubernetes.io/)

A production-grade GitOps deployment platform demonstrating automated continuous delivery on AWS EKS. This project showcases declarative infrastructure management, ArgoCD-driven deployments, and modern Kubernetes orchestration patterns.

**Platform:** AWS EKS  
**Infrastructure:** Terraform  
**Deployment:** ArgoCD GitOps  
**Orchestration:** Kubernetes

---

## Table of Contents

- [Overview](#overview)
- [GitOps Architecture](#gitops-architecture)
- [Infrastructure Components](#infrastructure-components)
- [Deployment Workflow](#deployment-workflow)
- [Prerequisites](#prerequisites)
- [Infrastructure Provisioning](#infrastructure-provisioning)
- [Application Deployment](#application-deployment)
- [ArgoCD Setup](#argocd-setup)
- [GitOps in Action](#gitops-in-action)
- [Troubleshooting](#troubleshooting)
- [Cost Considerations](#cost-considerations)
- [Related Projects](#related-projects)

---

## Overview

This platform demonstrates modern GitOps deployment patterns using ArgoCD on AWS EKS. The implementation showcases enterprise-grade practices for automated continuous delivery, declarative configuration management, and infrastructure as code.

### Core Capabilities

**GitOps Workflow:**
- Declarative application definitions stored in Git
- ArgoCD automatically syncs cluster state with Git repository
- Zero-manual-deployment pipeline from commit to production
- Complete audit trail of all infrastructure changes

**Container Orchestration:**
- AWS EKS manages Kubernetes control plane
- Automated pod scheduling and scaling
- Self-healing workload management
- Load-balanced service exposure

**Infrastructure Automation:**
- Terraform provisions entire EKS cluster and VPC
- Immutable infrastructure patterns
- Version-controlled infrastructure changes
- Reproducible environment creation

**Continuous Deployment:**
- Git commits trigger automatic synchronization
- Rolling updates with zero downtime
- Automated rollback on deployment failures
- Real-time deployment monitoring

### Business Value

**Operational Efficiency:**
- Deployment time reduced from hours to minutes
- Eliminated manual deployment errors
- Consistent deployments across environments
- Reduced mean time to recovery (MTTR)

**Developer Productivity:**
- Self-service deployments via Git commits
- No kubectl access required for deployments
- Clear visibility into deployment status
- Simplified rollback procedures

**Compliance & Security:**
- Complete audit trail in Git history
- Role-based access control (RBAC)
- Declarative security policies
- No direct cluster access needed

### Demo Application

The platform includes a lightweight Flask application that displays container metrics (CPU, memory, disk usage via `psutil`). This application serves as a **demonstration workload** to showcase:

- Container lifecycle management
- Service exposure via LoadBalancer
- Rolling update deployments
- ArgoCD synchronization behavior
- Kubernetes resource management

The Flask app is intentionally simple - the **real value** is in the GitOps deployment pipeline, not the application itself.

---

## GitOps Architecture

![Architecture Diagram](./architecture/diagram.png)

### Deployment Flow

```
Developer Workflow:
──────────────────

┌─────────────────┐
│  Developer      │
│  Git Commit     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  GitHub Repo    │
│  (Source)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ArgoCD         │
│  (Sync Engine)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  EKS Cluster    │
│  (Target State) │
└─────────────────┘
```

### Infrastructure Components

```
┌────────────────────────────────────────────────────────┐
│  AWS VPC (10.0.0.0/16)                                 │
│  ┌──────────────────────────────────────────────────┐ │
│  │  Public Subnets (Multi-AZ)                       │ │
│  │  ┌────────────────┐      ┌──────────────────┐   │ │
│  │  │ Application    │      │  ArgoCD          │   │ │
│  │  │ Load Balancer  │      │  Load Balancer   │   │ │
│  │  └───────┬────────┘      └────────┬─────────┘   │ │
│  └──────────┼────────────────────────┼─────────────┘ │
│             │                        │               │
│  ┌──────────▼────────────────────────▼─────────────┐ │
│  │  EKS Worker Nodes (Private Subnets)             │ │
│  │  ┌──────────────┐      ┌──────────────┐        │ │
│  │  │  App Pods    │      │ ArgoCD Pods  │        │ │
│  │  │  (Replicas)  │      │ (Controller) │        │ │
│  │  └──────────────┘      └──────────────┘        │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  ┌──────────────────────────────────────────────────┐ │
│  │  AWS ECR (Container Registry)                    │ │
│  │  Stores Docker images                            │ │
│  └──────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
```

### GitOps Synchronization Loop

**1. Developer Action:**
- Commit Kubernetes manifest changes to Git
- Update container image tags
- Modify deployment configuration

**2. ArgoCD Detection:**
- Polls Git repository every 3 minutes (default)
- Detects drift between Git and cluster state
- Triggers synchronization workflow

**3. Automated Deployment:**
- Applies new manifests to cluster
- Performs rolling update of pods
- Monitors deployment health
- Reports status back to ArgoCD UI

**4. Continuous Monitoring:**
- ArgoCD continuously compares desired vs. actual state
- Auto-heals configuration drift
- Alerts on sync failures
- Provides deployment history

---

## Infrastructure Components

### 1. Terraform Infrastructure (`terraform/`)

**VPC Configuration:**
- CIDR: `10.0.0.0/16`
- Multi-AZ public and private subnets
- Internet Gateway for public access
- NAT Gateway for private subnet egress

**EKS Cluster:**
- Managed Kubernetes control plane
- Node group with auto-scaling (min: 2, max: 4)
- Instance type: `t3.medium` (required for ArgoCD)
- Private endpoint access enabled

**IAM Configuration:**
- EKS cluster IAM role
- Node group IAM role with ECR access
- Service account roles for IRSA (IAM Roles for Service Accounts)

**Security:**
- Security groups for cluster and nodes
- RBAC configuration via aws-auth ConfigMap
- Network policies for pod isolation

### 2. Container Registry

**AWS ECR Repository:**
- Private repository: `my-cloud-app-repo`
- Image scanning enabled
- Lifecycle policies for cleanup
- Cross-region replication (optional)

**Image Management:**
- Tagged with Git commit SHA for traceability
- Automated builds via Docker
- Vulnerability scanning
- Immutable tags policy

### 3. Kubernetes Manifests (`manifests/`)

**Deployment (`deployment.yaml`):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-app
spec:
  replicas: 2  # High availability
  selector:
    matchLabels:
      app: monitoring-app
  template:
    spec:
      containers:
      - name: app
        image: <ECR_IMAGE_URL>:latest
        ports:
        - containerPort: 5000
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

**Service (`service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: monitoring-app-service
spec:
  type: LoadBalancer  # External access
  selector:
    app: monitoring-app
  ports:
  - port: 80
    targetPort: 5000
```

**ArgoCD Application (`argo-app.yaml`):**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/AkingbadeOmosebi/opsfolio-gitops-platform
    targetRevision: main
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## Deployment Workflow

### Complete GitOps Pipeline

**Stage 1: Code Development**
```bash
# Developer makes changes
vim app.py
git add .
git commit -m "feat: add memory usage endpoint"
git push origin main
```

**Stage 2: Container Build**
```bash
# Build and tag Docker image
docker build -t monitoring-app:$(git rev-parse --short HEAD) .

# Tag for ECR
docker tag monitoring-app:$(git rev-parse --short HEAD) \
  <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/my-cloud-app-repo:$(git rev-parse --short HEAD)

# Push to ECR
docker push <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/my-cloud-app-repo:$(git rev-parse --short HEAD)
```

**Stage 3: Manifest Update**
```bash
# Update deployment manifest with new image tag
vim manifests/deployment.yaml
# Change image: line to new tag
git add manifests/deployment.yaml
git commit -m "deploy: update to version $(git rev-parse --short HEAD)"
git push origin main
```

**Stage 4: ArgoCD Sync**
```
ArgoCD detects Git change
  ↓
Compares with cluster state
  ↓
Applies new manifests
  ↓
Rolling update begins
  ↓
Old pods drain gracefully
  ↓
New pods become ready
  ↓
Traffic switches to new pods
  ↓
Deployment complete
```

**Stage 5: Verification**
```bash
# Check deployment status
kubectl get deployments
kubectl get pods

# Check ArgoCD sync status
kubectl get applications -n argocd

# Access application
curl http://<LOAD_BALANCER_URL>
```

---

## Prerequisites

### Required Tools

- **AWS CLI:** >= 2.0, configured with credentials
- **Terraform:** >= 1.0.0
- **Docker:** >= 20.10
- **kubectl:** >= 1.24
- **Git:** For version control

### AWS Permissions

**IAM Policies Required:**
- `AmazonEKSClusterPolicy`
- `AmazonEKSWorkerNodePolicy`
- `AmazonEC2ContainerRegistryFullAccess`
- `AmazonVPCFullAccess`
- Custom policy for EKS node group creation

### Local Environment

```bash
# Verify installations
terraform version
docker --version
kubectl version --client
aws --version

# Configure AWS CLI
aws configure
# AWS Access Key ID: [your-key]
# AWS Secret Access Key: [your-secret]
# Default region name: us-east-1
# Default output format: json
```

---

## Infrastructure Provisioning

### Step 1: Create ECR Repository

```bash
# Create ECR repository
aws ecr create-repository \
  --repository-name my-cloud-app-repo \
  --region us-east-1

# Enable image scanning
aws ecr put-image-scanning-configuration \
  --repository-name my-cloud-app-repo \
  --image-scanning-configuration scanOnPush=true \
  --region us-east-1
```

### Step 2: Build and Push Docker Image

```bash
# Clone repository
git clone https://github.com/AkingbadeOmosebi/opsfolio-gitops-platform.git
cd opsfolio-gitops-platform

# Build Docker image
docker build -t monitoring-app:v1.0 .

# Authenticate to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

# Tag image
docker tag monitoring-app:v1.0 \
  <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-cloud-app-repo:v1.0

# Push to ECR
docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-cloud-app-repo:v1.0
```

### Step 3: Provision EKS Cluster with Terraform

```bash
cd terraform/

# Initialize Terraform
terraform init

# Review infrastructure plan
terraform plan

# Apply infrastructure
terraform apply

# Wait for cluster creation (~15-20 minutes)
```

**Resources Created:**
- VPC with public/private subnets
- EKS cluster with managed control plane
- Node group (2-4 t3.medium instances)
- Security groups and IAM roles
- Internet Gateway and NAT Gateway

### Step 4: Configure kubectl

```bash
# Update kubeconfig
aws eks update-kubeconfig \
  --region us-east-1 \
  --name <CLUSTER_NAME>

# Verify connection
kubectl get nodes
kubectl get namespaces
```

---

## Application Deployment

### Deploy Application to EKS

**Step 1: Update Manifest with ECR Image**

```bash
# Edit deployment.yaml
vim manifests/deployment.yaml

# Update image line:
# image: <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-cloud-app-repo:v1.0
```

**Step 2: Apply Kubernetes Manifests**

```bash
# Create deployment
kubectl apply -f manifests/deployment.yaml

# Create service
kubectl apply -f manifests/service.yaml

# Wait for pods to be ready
kubectl get pods -w

# Get LoadBalancer URL
kubectl get service monitoring-app-service
```

**Step 3: Verify Deployment**

```bash
# Check pod logs
kubectl logs -l app=monitoring-app

# Check pod status
kubectl describe pod <POD_NAME>

# Access application
LOAD_BALANCER_URL=$(kubectl get service monitoring-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$LOAD_BALANCER_URL
```

---

## ArgoCD Setup

### Install ArgoCD

**Step 1: Create ArgoCD Namespace**

```bash
kubectl create namespace argocd
```

**Step 2: Install ArgoCD**

```bash
# Apply ArgoCD manifests
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl get pods -n argocd -w
```

**Step 3: Expose ArgoCD Server**

```bash
# Patch service to LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get ArgoCD URL
kubectl get svc argocd-server -n argocd
```

**Step 4: Get Admin Password**

```bash
# Retrieve initial password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

**Step 5: Access ArgoCD UI**

```
URL: http://<ARGOCD_LOAD_BALANCER_URL>
Username: admin
Password: <password from previous step>
```

### Configure ArgoCD Application

**Step 1: Apply ArgoCD Application Manifest**

```bash
# Update argo-app.yaml with your repository URL
vim manifests/argo-app.yaml

# Apply manifest
kubectl apply -f manifests/argo-app.yaml
```

**Step 2: Verify Application in ArgoCD UI**

Navigate to ArgoCD UI:
- Applications → monitoring-app
- View sync status
- Check resource health
- Monitor deployment events

---

## GitOps in Action

### Scenario 1: Update Application Code

```bash
# Make code changes
vim app.py

# Build new image
docker build -t monitoring-app:v1.1 .
docker tag monitoring-app:v1.1 \
  <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-cloud-app-repo:v1.1
docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-cloud-app-repo:v1.1

# Update manifest
vim manifests/deployment.yaml
# Change image tag to v1.1

# Commit and push
git add .
git commit -m "deploy: update to v1.1"
git push origin main

# ArgoCD automatically syncs within ~3 minutes
# Watch in ArgoCD UI or:
kubectl get applications -n argocd -w
```

### Scenario 2: Scale Application

```bash
# Update replicas in manifest
vim manifests/deployment.yaml
# Change replicas: 2 → replicas: 4

# Commit and push
git add manifests/deployment.yaml
git commit -m "scale: increase replicas to 4"
git push origin main

# ArgoCD syncs automatically
# Verify scaling
kubectl get pods -l app=monitoring-app
```

### Scenario 3: Rollback Deployment

**Option 1: Via Git**
```bash
# Revert to previous commit
git revert HEAD
git push origin main

# ArgoCD syncs to previous state
```

**Option 2: Via ArgoCD UI**
- Navigate to Application
- History tab
- Select previous revision
- Click "Sync"

**Option 3: Via kubectl**
```bash
# Rollback deployment
kubectl rollout undo deployment/monitoring-app

# Note: This creates drift - ArgoCD will detect and may re-sync
```

---

## Troubleshooting

### Common Issues

**Issue 1: ArgoCD Dex Pod CrashLoopBackOff**

**Symptom:**
```bash
kubectl get pods -n argocd
# argocd-dex-server-xxx  0/1  CrashLoopBackOff
```

**Root Cause:** Insufficient node resources (t3.small too small)

**Solution:**
```bash
# Update node instance type in terraform/main.tf
resource "aws_eks_node_group" "main" {
  instance_types = ["t3.medium"]  # Changed from t3.small
}

# Apply changes
terraform apply

# Wait for node replacement
kubectl get nodes -w
```

---

**Issue 2: kubectl Unauthorized Error**

**Symptom:**
```bash
kubectl get nodes
# error: You must be logged in to the server (Unauthorized)
```

**Root Cause:** IAM user not in aws-auth ConfigMap

**Solution:**
```bash
# Edit aws-auth ConfigMap
kubectl edit configmap aws-auth -n kube-system

# Add your IAM user under mapUsers:
mapUsers: |
  - userarn: arn:aws:iam::<ACCOUNT_ID>:user/<YOUR_USER>
    username: <YOUR_USER>
    groups:
      - system:masters

# Save and exit
```

---

**Issue 3: Application LoadBalancer Pending**

**Symptom:**
```bash
kubectl get service monitoring-app-service
# EXTERNAL-IP: <pending>
```

**Root Cause:** AWS Load Balancer Controller not installed or subnet tags missing

**Solution:**
```bash
# Check public subnets have correct tags
aws ec2 describe-subnets --filters "Name=tag:Name,Values=*public*"

# Required tags:
# kubernetes.io/role/elb = 1
# kubernetes.io/cluster/<CLUSTER_NAME> = shared

# Add tags via Terraform or AWS console
```

---

**Issue 4: ArgoCD Not Syncing**

**Symptom:** Changes pushed to Git but ArgoCD shows "Synced" with old state

**Root Cause:** ArgoCD cache or sync interval

**Solution:**
```bash
# Manual sync via CLI
argocd app sync monitoring-app

# Or force refresh
argocd app get monitoring-app --refresh

# Check sync policy in argo-app.yaml
# Ensure automated sync is enabled
```

---

**Issue 5: Image Pull Errors**

**Symptom:**
```bash
kubectl describe pod <POD_NAME>
# Events: Failed to pull image: access denied
```

**Root Cause:** Node IAM role lacks ECR permissions

**Solution:**
```bash
# Verify node role has ECR policy
aws iam list-attached-role-policies \
  --role-name <NODE_ROLE_NAME>

# Attach ECR policy if missing
aws iam attach-role-policy \
  --role-name <NODE_ROLE_NAME> \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
```

---

## Cost Considerations

### Monthly Cost Breakdown (us-east-1)

| Service | Specification | Estimated Cost |
|---------|--------------|----------------|
| **EKS Control Plane** | $0.10/hour | ~$73 |
| **EC2 Instances** | 2× t3.medium (730 hrs) | ~$60 |
| **Application Load Balancer** | 1 ALB + data transfer | ~$20 |
| **NAT Gateway** | 1 NAT + data transfer | ~$35 |
| **ECR Storage** | < 10 GB images | ~$1 |
| **Data Transfer** | Inter-AZ + egress | ~$10 |
| **Total** | | **~$199/month** |

### Cost Optimization Strategies

**Compute Savings:**
- Use EC2 Spot Instances for non-production (70% savings)
- Right-size node instance types based on workload
- Implement cluster autoscaler to scale down during low usage
- Consider Fargate for specific workloads (pay per pod)

**Network Savings:**
- Remove NAT Gateway if no private subnet egress needed
- Use VPC endpoints for AWS service access
- Minimize cross-AZ data transfer
- Use CloudFront for static content delivery

**Storage Savings:**
- Implement ECR lifecycle policies (delete old images)
- Use EBS volume snapshots efficiently
- Clean up unused persistent volumes

**Monitoring:**
- Set up AWS Budgets with $200 threshold
- Enable Cost Anomaly Detection
- Review Cost Explorer weekly
- Tag resources for cost allocation

---

## Project Structure

```
opsfolio-gitops-platform/
├── app.py                    # Flask application
├── requirements.txt          # Python dependencies
├── Dockerfile               # Container definition
├── manifests/               # Kubernetes manifests
│   ├── deployment.yaml      # Application deployment
│   ├── service.yaml         # LoadBalancer service
│   └── argo-app.yaml        # ArgoCD application
├── terraform/               # Infrastructure as Code
│   ├── main.tf              # Main configuration
│   ├── variables.tf         # Input variables
│   ├── outputs.tf           # Output values
│   ├── provider.tf          # AWS provider
│   └── eks.tf               # EKS cluster config
├── architecture/            # Documentation
│   └── diagram.png          # Architecture diagram
├── templates/               # Flask templates
│   └── index.html           # Application UI
├── static/                  # Static assets
│   ├── css/
│   └── js/
├── .gitignore
└── README.md
```

---

## Related Projects

Part of the **Opsfolio** infrastructure series:

**[Opsfolio: Kubernetes Platform](https://github.com/AkingbadeOmosebi/Opsfolio-Interview-App)**  
DevSecOps platform with 8-layer security pipeline, ArgoCD, and Prometheus monitoring

**[Opsfolio: Cross-Cloud Platform](https://github.com/AkingbadeOmosebi/opsfolio-crosscloud-platform)**  
Multi-cloud integration with AWS ECR + Azure Container Apps and OIDC federation

**[Opsfolio: Resilience Platform](https://github.com/AkingbadeOmosebi/opsfolio-resilience-platform)**  
High-availability infrastructure with Multi-AZ ECS and RDS automated failover

---

## Key Learnings

### Technical Insights

**GitOps Benefits Realized:**
- Deployment consistency across environments
- Complete audit trail of changes
- Self-service deployments for developers
- Simplified rollback procedures

**Infrastructure Challenges:**
- EKS node sizing critical for ArgoCD stability
- IAM RBAC configuration requires careful planning
- LoadBalancer costs can accumulate quickly
- Private subnet architecture adds complexity

**Best Practices Established:**
- Image tagging with Git commit SHAs for traceability
- Resource requests/limits prevent node resource exhaustion
- Automated sync with self-heal for consistency
- Namespace isolation for multi-tenancy

### Production Recommendations

**For Real-World Deployments:**
- Implement network policies for pod-to-pod security
- Use External Secrets Operator for sensitive data
- Configure pod disruption budgets for availability
- Implement horizontal pod autoscaling
- Add Prometheus monitoring and alerting
- Use ingress controller instead of multiple LoadBalancers
- Implement backup strategy for cluster state

---

## Contact

**Akingbade Omosebi**  
Platform Engineer | DevOps Specialist | Berlin, Germany

Specializing in GitOps workflows, Kubernetes orchestration, and automated deployment pipelines.

- **GitHub:** [github.com/AkingbadeOmosebi](https://github.com/AkingbadeOmosebi)
- **LinkedIn:** [linkedin.com/in/aomosebi](https://linkedin.com/in/aomosebi)
- **Technical Blog:** [dev.to/akingbade_omosebi](https://dev.to/akingbade_omosebi)

---

## License

This project is open source and available for educational purposes. Feel free to use as reference for implementing GitOps workflows in your own infrastructure.

---

**Built with GitOps principles | Part of the Opsfolio infrastructure series**
