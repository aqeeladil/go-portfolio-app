# Portfolio Website | End-to-end CICD Implementation using Docker, Aws-Eks, Github Actions, Helm, ArgoCD & Ingress Conroller.

This repository demonstrates a complete end-to-end DevOps workflow for a Golang web application, implementing industry best practices. It is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests. Below is a detailed guide for setting up and executing each step.

## Features Implemented

1. **Containerization**: Multi-stage Docker build for optimized images.
  - he Multi-stage build is a feature of Docker that allows you to use multiple build stages in a single Dockerfile. 
  - This will reduce the size of the final Docker image and also secure the image by removing unnecessary files and packages. 

2. **Kubernetes Deployment**: YAML manifests for deployment, services, and ingress.

3. **CI/CD Pipeline**: 
  - Continuous Integration (CI) using GitHub Actions.
  - Continuous Delivery (CD) with Argo CD.

4. **Cluster Management**: Setup of an Amazon EKS cluster.

5. **Helm Charts**: Reusable Helm charts for multiple environments.

6. **Ingress and DNS Mapping**: Expose the application through an ingress controller and DNS.

## Summary Diagram
![image](https://github.com/user-attachments/assets/45f4ef12-c5b5-4247-9d43-356b5dfb671b)

## Repository Structure

```
├── Dockerfile                # Multi-stage Docker build file
├── k8s-manifests
│   ├── deployment.yaml       # Deployment manifest
│   ├── service.yaml          # Service manifest
│   ├── ingress.yaml          # Ingress manifest
├── helm/
│   ├── templates/            # Kubernetes manifests with variables
│   └── values.yaml           # Parameterized values
├── .github/
│   └── workflows/
│       └── ci.yaml           # GitHub Actions workflow
├── gitops/argocd
│   ├── application.yaml            # ArgoCD application
│   
```

## Prerequisites

```bash
# Launch an Ec2 Ubuntu Instance
# Configure security groups: Allow ports: 22, 80, 8080.
# Create an IAM Role and attach the Role to the EC2 Instance.
# SSH into your instance
ssh -i "path/to/your-key.pem" ubuntu@<instance-public-ip>

# Update and Install Sysyte Packages
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget apt-transport-https gnupg software-properties-common

# Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
# Log out and log back in to apply the Docker group change.

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install eksctl
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
# Configure AWS CLI with your credentials:
aws configure
# Enter the details: Access Key ID, Secret Access Key, Default region (e.g., us-east-1, Output format (e.g., json)

# Install Helm
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-add-repository "deb https://baltocdn.com/helm/stable/debian/ all main"
sudo apt update && sudo apt install -y helm
helm version
```

## Steps in Detail

### 1. Understanding the Application

1. **Clone the repository:**
  ```bash
  git clone https://github.com/aqeeladil/go-portfolio-app.git
  cd go-portfolio-app
  ```

2. **Build and run locally:**
  ```bash
  go build -o main .
  ls
  ./main
  ```

3. **Access the application** at `<ec2-instance-ip>:8080/courses`

### 2. Containerization

- **Objective**: Package the Go application into a Docker image.

- **Steps**:

  1. Create a **multi-stage Dockerfile**:
     - **Stage 1**: Use the `golang` base image to compile the application.
     - **Stage 2**: Use a distroless image for enhanced security and reduced image size.

  2. **Build, test & push the Docker image** to Docker Hub or a container registry.
      ```bash
      docker build -t aqeeladil/go-portfolio-app:v1 .
      docker run -p 8080:8080 -it aqeeladi/go-portfolio-app:v1
      docker push aqeeladi/go-portfolio-app:v1
      ```

### 3. EKS Cluster Setup

**Tools Required**:
- `eksctl`: For EKS cluster setup.
- `kubectl`: For Kubernetes management.
- AWS CLI: For authentication.

**Setup Command**:
```bash
eksctl create cluster --name go-portfolio-app-cluster --region us-east-1

# The creation process takes about 10–15 minutes. During this time, eksctl setsup: The EKS control plane, Managed worker nodes, Networking components (VPC,subnets, etc).

kubectl get nodes
```

### 4. Kubernetes Deployment

- **Files Created**:
  - `deployment.yaml`: Defines pods and replicas.
  - `service.yaml`: Exposes the application internally.
  - `ingress.yaml`: Configures external access to the application.

- **Best Practices**:
  - Use consistent labels and selectors.
  - Define resource limits for pods.

**Deploy on Kubernetes**:
```bash
kubectl apply -f k8s-manifests/deployment.yaml
kubectl apply -f k8s-manifests/service.yaml
kubectl apply -f k8s-manifests/ingress.yaml

# View cluster resources
kubectl get all    
kubectl get ing
kubectl get svc

# Optional Step
kubectl edit svc go-web-app
# Change the type ClusterIP to NodePort
kubectl get svc
# Copy the NodePort address (e.g; 80:32099/TCP)
kubectl get nodes -o wide
# Copy any External IP address (e.g; 54:161:25:151)
# Open the browser at <External-IP>:NodePort/courses (e.g; 54:161:25:151:32099/courses)

# Install the NGINX Ingress Controller.
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
kubectl get pods -n ingress-nginx

# Configure host-based routing in the ingress resource.
# Map the ingress IP address to a domain name using DNS.
# Update `/etc/hosts`
kubectl get ing
# Copy the address
nslookup <loadbalanacer-address>
# copy the ingress-ip-address
sudo vim /etc/hosts
# Add the following line: 
<Ingress-IP> go-portfolio-app.local

# Visit http://go-portfolio-app.local/courses in your browser.              
```

### 5. Helm Charts

- **Why Helm?**: Simplifies deployment across environments by using reusable and parameterized templates.

- **Structure**:
  - `values.yaml`: Contains environment-specific values (e.g., image tags, replica counts).
  - `templates/`: Stores Kubernetes manifests with placeholders for variables.

**Command**:
```bash
mkdir helm && cd helm
helm create go-portfolio-app-chart

# Verify 
kubectl get all
kubectl delete deploy go-portfolio-app
kubectl delete svc go-portfolio-app
kubectl delete ing go-portfolio-app

# Deploy the application using Helm.
helm install go-portfolio-app ./go-portfolio-app-chart 
helm upgrade go-portfolio-app ./go-portfolio-app-chart 

# Verify again
kubectl get deployment
kubectl get svc
kubectl get ing
helm uninstall go-portfolio-app
kubectl get all
```

- **Update `values.yaml`** file with your configurations.

### 6. CI/CD with GitHub Actions and Argo CD

#### CI Pipeline (GitHub Actions)

- **Set up `.github/workflows/ci.yaml`**.

- **Stages**:
  1. **Build and Test**: Compile and test the code.
  2. **Static Code Analysis**: Use `golangci-lint` to check code quality.
  3. **Docker Image Build**: Create and push the Docker image.
  4. **Helm Update**: Update `values.yaml` with the new image tag.

- **Trigger**: Executes on every push to the `main` branch.

- **Configure Secrets:**
  - Create secrets for `TOKEN`, `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`
  - Github Repo > `Settings` > `Secrets & Variables` > `Actions` > `New Repository Secrets`.
  - For Dockerhub credentials, visit your account settings.
  - For Github profile Access Token, go to your profile `Settings` > `Developer Settings` > `Personal Access Token` > `Token Classic` > `Generate New Token`.

#### CD with Argo CD

Automatically detects changes in the Helm chart and deploys them to the EKS cluster. Supports self-healing to revert manual changes in the cluster.

**Install Argo CD**:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cstable/manifests/install.yaml
```

**Access Argo CD UI using Loadbalancer Service**:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get the Loadbalancer service IP
kubect get svc -n argocd
kubectl get svc argocd-server -n argocd
# Copy the <External-ip> and paste it in a browser to access Argocd UI.

# If it takes time. You can alternatively access Argocd using NodePort
# Copy the nodeport address of the argocd-server service
kubectl get nodes -o wide
# Copy the <External-ip> 
# Access Argocd at <External-IP>:<Nodeport>
```

**Default login:**
- Username: `admin`
- Password: Retrieve from:
```bash
kubectl get secrets -n argocd
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

**Apply the resource to create argocd Application**:
```bash
kubectl apply -f gitops/argocd/application.yaml
```

## End-to-End Workflow

1. **Code Change**: 
    - Make a change to the application code.
    - Commit and push to main branch

2. **CI Pipeline**:
   - Runs build, tests, and static analysis.
   - Creates a Docker image and updates the Helm chart with a new tag.

3. **CD Pipeline**:
   - Argo CD detects the updated Helm chart.
   - Deploys the changes to the EKS cluster.

4. **Ingress Controller**:
   - Routes traffic based on DNS and host configurations.



