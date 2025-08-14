# Batch-11 


### 🎯 What You'll Learn
- **Docker**: Multi-stage builds, container orchestration with Docker Compose
- **Kubernetes**: Pod management, services, deployments, ingress controllers
- **Helm**: Chart creation, templating, package management
- **Terraform**: AWS infrastructure provisioning, state management, modular design

---

## 📋 Table of Contents

- [🐳 Docker](#-docker)
- [☸️ Kubernetes](#️-kubernetes)
- [⛵ Helm](#-helm)
- [🏗️ Terraform](#️-terraform)
- [📁 Project Structure](#-project-structure)
- [🔧 Troubleshooting](#-troubleshooting)
- [📚 Additional Resources](#-additional-resources)

---

## 🐳 Docker

### 📦 Prerequisites

#### Install Docker
```bash
# Windows (using Chocolatey)
choco install docker-desktop

# macOS (using Homebrew)
brew install --cask docker

# Linux (Ubuntu/Debian)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

#### Install Docker Compose
```bash
# Linux
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version
docker-compose --version
```

### 🏗️ Images Overview

#### Alpine Apache Server
- **Base Image**: Alpine Linux (lightweight ~5MB)
- **Web Server**: Apache HTTP Server
- **Port**: 80
- **Features**: Custom index.html, production-ready configuration

#### Tomcat Application Server
- **Base Image**: OpenJDK 17 JDK Slim
- **Application Server**: Apache Tomcat 9.0.107
- **Port**: 8080
- **Features**: Multi-stage build, volume mounting for applications

#### Database Server
- **Image**: MariaDB
- **Port**: 3306
- **Credentials**: root/admin123

### 🔨 Building and Running

#### Build Individual Images
```bash
# Build Apache image with custom tag
docker build -t surajbele/batch-11:alpine-apache2 -f docker/Dockerfile docker/

# Build Tomcat image
docker build -t surajbele/batch-11:tomcat_server -f docker/dockerfile-tomcat docker/

# Verify images
docker images | grep surajbele
```

#### Using Docker Compose
```bash
# Start all services in detached mode
docker-compose -f docker/compose.yaml up -d

# View running containers
docker-compose -f docker/compose.yaml ps

# View logs
docker-compose -f docker/compose.yaml logs -f websesrver
docker-compose -f docker/compose.yaml logs -f tomcatserver

# Scale services
docker-compose -f docker/compose.yaml up -d --scale websesrver=3

# Stop all services
docker-compose -f docker/compose.yaml down

# Stop and remove volumes
docker-compose -f docker/compose.yaml down -v
```

#### Container Management
```bash
# Execute commands inside containers
docker exec -it <container_name> /bin/sh

# Copy files to/from containers
docker cp index.html <container_name>:/var/www/localhost/htdocs/

# Monitor resource usage
docker stats

# Inspect container details
docker inspect <container_name>
```

### 🔍 Testing Your Deployment
```bash
# Test Apache server
curl http://localhost:80

# Test Tomcat server
curl http://localhost:8080

# Test MariaDB connection
mysql -h localhost -P 3306 -u root -padmin123
```

---

## ☸️ Kubernetes

### 📦 Prerequisites

#### Install kubectl
```bash
# Windows (using Chocolatey)
choco install kubernetes-cli

# macOS (using Homebrew)
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

#### Set up a Kubernetes Cluster

##### Option 1: Minikube (Local Development)
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start --driver=docker

# Enable ingress addon
minikube addons enable ingress

# Get cluster info
kubectl cluster-info
```

##### Option 2: Docker Desktop (Windows/macOS)
```bash
# Enable Kubernetes in Docker Desktop settings
# Then verify
kubectl config current-context
```

### 🚀 Deployment Strategies

#### Deploy Individual Resources
```bash
# Deploy pods
kubectl apply -f k8s/pod.yaml

# Deploy services
kubectl apply -f k8s/service.yaml

# Deploy deployments
kubectl apply -f k8s/deployment.yaml

# Check deployment status
kubectl rollout status deployment/nginx-deployment
```

#### Deploy All Resources
```bash
# Apply all manifests
kubectl apply -f k8s/ --recursive

# Watch resources being created
kubectl get all -w
```

### 📊 Monitoring and Management

#### Resource Inspection
```bash
# Get all resources
kubectl get all -o wide

# Describe specific resources
kubectl describe pod <pod-name>
kubectl describe service <service-name>
kubectl describe deployment <deployment-name>

# Get resource YAML
kubectl get pod <pod-name> -o yaml
```

#### Logs and Debugging
```bash
# View pod logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow logs

# Execute commands in pods
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding for testing
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80
```

#### Scaling and Updates
```bash
# Scale deployments
kubectl scale deployment nginx-deployment --replicas=5

# Rolling updates
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1

# Check rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment
```

### 🌐 Ingress Configuration

#### Enable Ingress Controller
```bash
# For Minikube
minikube addons enable ingress

# For cloud providers, install NGINX ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

#### Deploy Ingress Resources
```bash
# Apply ingress configuration
kubectl apply -f k8s/ingress/

# Check ingress status
kubectl get ingress
kubectl describe ingress myingress

# Get ingress IP (for Minikube)
minikube ip
```

#### Testing Ingress
```bash
# Add entries to /etc/hosts (Linux/macOS) or C:\Windows\System32\drivers\etc\hosts (Windows)
echo "$(minikube ip) tar.com tar.mobile.com" | sudo tee -a /etc/hosts

# Test routing
curl http://tar.com
curl http://tar.mobile.com
```

### 📦 Working with ConfigMaps and Secrets
```bash
# Create ConfigMap
kubectl create configmap app-config --from-file=config.properties

# Create Secret
kubectl create secret generic app-secret --from-literal=password=mysecretpassword

# Apply from files
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrete.yaml
```

---

## ⛵ Helm

### 📦 Prerequisites

#### Install Helm
```bash
# Windows (using Chocolatey)
choco install kubernetes-helm

# macOS (using Homebrew)
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify installation
helm version
```

### 🎯 Working with the Spiderman Chart

#### Chart Structure Analysis
```
k8s/helmchart/spiderman/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
└── templates/
    ├── _helpers.tpl     # Template helpers
    ├── deployment.yaml  # Deployment template
    ├── service.yaml     # Service template
    └── ingress.yaml     # Ingress template
```

#### Chart Operations
```bash
# Validate chart syntax
helm lint k8s/helmchart/spiderman

# Dry run to see generated manifests
helm install spiderman k8s/helmchart/spiderman --dry-run --debug

# Package the chart
helm package k8s/helmchart/spiderman

# Install the chart
helm install spiderman k8s/helmchart/spiderman

# List installed releases
helm list

# Get release status
helm status spiderman

# Upgrade with custom values
helm upgrade spiderman k8s/helmchart/spiderman --set replicaCount=3

# Upgrade with values file
helm upgrade spiderman k8s/helmchart/spiderman -f custom-values.yaml
```

#### Advanced Helm Operations
```bash
# Show chart values
helm show values k8s/helmchart/spiderman

# Get release values
helm get values spiderman

# Rollback to previous version
helm rollback spiderman 1

# Uninstall release
helm uninstall spiderman

# History of releases
helm history spiderman
```

#### Custom Values Example
Create `custom-values.yaml`:
```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.20"

service:
  type: LoadBalancer
  port: 80

ingress:
  enabled: true
  hosts:
    - host: my-app.local
      paths:
        - path: /
          pathType: Prefix
```

#### Repository Management
```bash
# Add Helm repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# Update repositories
helm repo update

# Search for charts
helm search repo nginx

# Install from repository
helm install my-nginx bitnami/nginx
```

---

## 🏗️ Terraform

### 📦 Prerequisites

#### Install Terraform
```bash
# Windows (using Chocolatey)
choco install terraform

# macOS (using Homebrew)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (Ubuntu/Debian)
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify installation
terraform version
```

#### Configure AWS Credentials
```bash
# Install AWS CLI
pip install awscli

# Configure credentials
aws configure
# Enter: Access Key ID, Secret Access Key, Region (us-east-1), Output format (json)

# Or set environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"

# Verify credentials
aws sts get-caller-identity
```

### 🏛️ Infrastructure Components

#### Current Terraform Configuration
- **VPC Module**: Custom VPC with public/private subnets
- **Security Groups**: HTTP (80) and SSH (22) access
- **EC2 Instance**: t2.micro with custom AMI
- **S3 Backend**: Remote state storage
- **Modular Design**: Separate VPC and instance modules

#### Terraform Workflow
```bash
# Navigate to terraform directory
cd terraform

# Initialize Terraform (download providers, set up backend)
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt

# Plan infrastructure changes
terraform plan

# Apply changes with auto-approval
terraform apply -auto-approve

# Show current state
terraform show

# List resources in state
terraform state list

# Destroy infrastructure
terraform destroy -auto-approve
```

#### Working with Variables
```bash
# Override variables via command line
terraform plan -var="instance_type=t2.small" -var="region=us-west-2"

# Use variables file
terraform plan -var-file="prod.tfvars"

# Set via environment variables
export TF_VAR_instance_type="t2.small"
terraform plan
```

#### State Management
```bash
# Show state
terraform show

# Import existing resource
terraform import aws_instance.myinstance i-1234567890abcdef0

# Remove resource from state (without destroying)
terraform state rm aws_instance.myinstance

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Refresh state
terraform refresh
```

#### Module Development
```bash
# Initialize module directory
mkdir -p modules/vpc
cd modules/vpc

# Create module files
touch main.tf variables.tf outputs.tf

# Use module in root configuration
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr = var.vpc_cidr
  environment = var.environment
}
```

### 🔒 Best Practices

#### Security
```bash
# Use terraform.tfvars for sensitive data (add to .gitignore)
echo "*.tfvars" >> .gitignore

# Encrypt state file
terraform {
  backend "s3" {
    bucket         = "batch-11-terraform"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

#### Cost Optimization
- Use `t2.micro` instances (free tier eligible)
- Implement auto-shutdown for development environments
- Use spot instances for non-critical workloads

---

## 📁 Project Structure

```
📦 batch-11/
├── 📁 docker/                    # Container definitions
│   ├── 🐳 Dockerfile            # Alpine + Apache setup
│   ├── 🐳 dockerfile-tomcat     # Multi-stage Tomcat build
│   ├── 🔧 compose.yaml          # Multi-service orchestration
│   └── 📄 index.html            # Custom web content
├── 📁 k8s/                      # Kubernetes manifests
│   ├── 🚀 deployment.yaml       # Nginx deployment (3 replicas)
│   ├── 🌐 service.yaml          # NodePort & LoadBalancer services
│   ├── 🎯 pod.yaml              # Multi-container pod examples
│   ├── 📋 configmap.yaml        # Configuration management
│   ├── 🔐 secrete.yaml          # Secrets management
│   ├── 🛡️ daemonset.yaml        # Node-level services
│   ├── 📊 replicaSet.yaml       # Pod replication
│   ├── 💾 statefull.yaml        # Stateful applications
│   ├── 📁 ingress/              # Ingress configurations
│   │   ├── 🌐 ingress.yaml      # Host-based routing
│   │   ├── 🚀 deployment.yaml   # Ingress-specific deployments
│   │   └── 🌐 services.yaml     # Ingress-specific services
│   ├── 📁 helmchart/            # Helm package management
│   │   └── 📁 spiderman/        # Custom Helm chart
│   │       ├── 📋 Chart.yaml    # Chart metadata
│   │       ├── ⚙️ values.yaml   # Default values
│   │       └── 📁 templates/    # Kubernetes templates
│   │           ├── 🚀 deployment.yaml
│   │           ├── 🌐 service.yaml
│   │           └── 🌐 ingress.yaml
│   ├── 📁 rbac/                 # Role-based access control
│   │   ├── 🔐 ironman.crt       # SSL certificate
│   │   ├── 🔐 ironman.csr       # Certificate signing request
│   │   ├── 🔐 ironman.key       # Private key
│   │   ├── 🛡️ role.yaml         # RBAC roles
│   │   └── 🔗 rolebinding.yaml  # Role bindings
│   └── 📁 volume/               # Persistent storage
│       ├── 🎯 pod.yaml          # Pod with volume mounts
│       ├── 💾 pv.yaml           # Persistent volume
│       └── 📋 pvc.yaml          # Persistent volume claim
├── 📁 terraform/                # Infrastructure as Code
│   ├── 🏗️ main.tf              # Primary configuration
│   ├── 📊 output.tf             # Output definitions
│   ├── ⚙️ variable.tf           # Variable declarations
│   ├── 🔧 terraform.tfvars      # Variable values
│   ├── 📁 instance/             # EC2 instance module
│   │   ├── 🏗️ main.tf
│   │   ├── 📊 output.tf
│   │   └── ⚙️ variable.tf
│   └── 📁 vpc/                  # VPC module
│       ├── 🏗️ main.tf
│       ├── 📊 output.tf
│       └── ⚙️ variable.tf
└── 📚 README.md                 # This comprehensive guide
```

---

## 🔧 Troubleshooting

### Docker Issues

#### Container Won't Start
```bash
# Check container logs
docker logs <container_name>

# Inspect container configuration
docker inspect <container_name>

# Check if port is already in use
netstat -tulpn | grep :80
lsof -i :80  # macOS/Linux
```

#### Build Failures
```bash
# Clear Docker cache
docker system prune -a

# Build with no cache
docker build --no-cache -t myimage .

# Check Dockerfile syntax
docker build --dry-run -t myimage .
```

### Kubernetes Issues

#### Pod Stuck in Pending
```bash
# Check node resources
kubectl describe nodes

# Check pod events
kubectl describe pod <pod-name>

# Check resource quotas
kubectl describe resourcequota
```

#### Service Not Accessible
```bash
# Check service endpoints
kubectl get endpoints

# Test service from within cluster
kubectl run debug --image=busybox --rm -it -- sh
nslookup <service-name>

# Check network policies
kubectl get networkpolicies
```

#### Ingress Not Working
```bash
# Check ingress controller status
kubectl get pods -n ingress-nginx

# Verify ingress configuration
kubectl describe ingress <ingress-name>

# Check DNS resolution
nslookup tar.com
```

### Helm Issues

#### Chart Validation Errors
```bash
# Lint chart
helm lint k8s/helmchart/spiderman

# Template validation
helm template spiderman k8s/helmchart/spiderman

# Debug template rendering
helm install spiderman k8s/helmchart/spiderman --debug --dry-run
```

#### Release Failures
```bash
# Check release status
helm status <release-name>

# View release history
helm history <release-name>

# Get release manifest
helm get manifest <release-name>
```

### Terraform Issues

#### State Lock Issues
```bash
# Force unlock (use carefully)
terraform force-unlock <lock-id>

# Check state status
terraform state pull
```

#### Provider Issues
```bash
# Re-initialize providers
terraform init -upgrade

# Clear provider cache
rm -rf .terraform/
terraform init
```

#### Resource Conflicts
```bash
# Import existing resources
terraform import aws_instance.myinstance i-1234567890abcdef0

# Show differences
terraform plan -detailed-exitcode
```

---

## 🔐 Security Best Practices

### Docker Security
- Use non-root users in containers
- Scan images for vulnerabilities
- Use minimal base images (Alpine)
- Keep secrets out of Dockerfiles

### Kubernetes Security
- Enable RBAC
- Use network policies
- Scan container images
- Implement pod security policies

### Terraform Security
- Store state remotely with encryption
- Use IAM roles instead of access keys
- Enable CloudTrail for audit logging
- Implement resource tagging

---

## 📚 Additional Resources

### Documentation
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
- [Helm Chart Development](https://helm.sh/docs/chart_template_guide/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

### Tools and Extensions
- **VS Code Extensions**: Docker, Kubernetes, Terraform
- **CLI Tools**: k9s, kubectx, terraform-docs, tflint
- **Monitoring**: Prometheus, Grafana, ELK Stack

### Learning Platforms
- [Katacoda](https://katacoda.com/) - Interactive learning scenarios
- [Play with Docker](https://labs.play-with-docker.com/) - Browser-based Docker playground
- [Play with Kubernetes](https://labs.play-with-k8s.com/) - Browser-based K8s playground

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Happy DevOps-ing! 🚀**