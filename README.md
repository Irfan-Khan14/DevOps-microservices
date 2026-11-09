# 🚀 AWS EKS Microservices Platform (Driven by GitOps)

Welcome to my cloud playground! This repository houses the infrastructure and deployment code for a fully automated, self-healing microservices environment on AWS. 

Instead of deploying things manually, I built this project to act as a **GitOps machine**. That means this repository isn't just storing code—it *is* the single source of truth for the entire cloud environment. If it's not in Git, it doesn't exist in the cluster.

---

## 📖 The Story Behind the Project
I built this platform to get hands-on with the exact tools used in modern production environments. I wanted to move beyond just running containers locally and experience the entire lifecycle: provisioning the cloud, setting up a CI/CD pipeline, and automating deployments so the system practically runs itself.

**The Goal:** Push a code change to GitHub, grab a coffee, and watch it automatically deploy to a secure, monitored Kubernetes cluster on AWS without ever running a manual `kubectl` command.

---

## 🧠 How It Works (The Architecture)

Here is a plain-English breakdown of what happens under the hood:

### 1. The Foundation (Terraform)
Before we can run apps, we need a place to put them. I used **Terraform** to act as the architect. It talks to AWS and builds a secure virtual network (VPC), sets up public and private subnets, and provisions an **Amazon EKS (Kubernetes)** cluster with dedicated worker nodes. 

### 2. The Delivery Pipeline (GitHub Actions)
Whenever new application code is pushed to the `main` branch, **GitHub Actions** wakes up. It acts as the factory: 
* It packages the app into a Docker container.
* It stamps the container with a unique ID (the Git commit SHA).
* It securely ships that container to our private storage vault (**Amazon ECR**).
* Finally, it updates the Kubernetes blueprints in this repository to point to the new container.

### 3. The Watchdog (ArgoCD & GitOps)
This is where the magic happens. **ArgoCD** lives inside the Kubernetes cluster and constantly watches this GitHub repository. 
* If it notices the blueprints here have changed (thanks to our CI pipeline), it immediately syncs the live AWS cluster to match. 
* **Self-Healing:** If someone accidentally deletes a running application in the cloud, ArgoCD says, *"Wait, the GitHub blueprint says this should be running,"* and instantly recreates it.

### 4. The Dashboard (Prometheus & Grafana)
You can't manage what you can't measure. I deployed a monitoring stack that constantly checks the cluster's heartbeat. **Prometheus** collects the data (CPU, memory, network traffic), and **Grafana** turns that data into beautiful, readable charts.

---

## 🛠️ The Technology Stack

* **The Cloud:** AWS (EKS, EC2, VPC, IAM, ECR)
* **Infrastructure as Code:** Terraform
* **Containers & Orchestration:** Docker, Kubernetes
* **CI/CD:** GitHub Actions (CI), ArgoCD (CD)
* **Observability:** Prometheus, Grafana
* **The App:** A multi-tier Node.js & PostgreSQL e-commerce backend

---

## 📂 Repository Tour

I've kept the folders organized exactly by their function:

```text
├── terraform/                # The AWS infrastructure code (VPC, EKS, Node Groups)
├── .github/workflows/        # The CI pipeline instructions for GitHub Actions
├── kubernetes/               # The brain of the GitOps operation
│   ├── app/                  # Blueprints for the microservices (Frontend, API, Auth)
│   ├── monitoring/           # Blueprints for Prometheus and Grafana
│   └── argocd/               # Instructions telling ArgoCD how to monitor this repo
└── src/                      # The actual application source code
🚧 Lessons Learned & Challenges Overcome
Let's be real—building this wasn't just a breeze. Here are a few things I tackled along the way:

The AWS Authentication Maze: Getting the AWS Console to talk to the Kubernetes Role-Based Access Control (RBAC) was a massive headache. The cluster was using the legacy aws-auth ConfigMap, and manual edits in Notepad kept corrupting the YAML formatting. I solved this by piping machine-perfect YAML directly into the cluster via the terminal, successfully granting my IAM user system:masters access.

Stateful Databases in Kubernetes: Running a stateless app is easy; running a database is hard. I had to figure out how to seed a PostgreSQL database with initial data automatically. I solved this by writing a Kubernetes Job that runs on startup to populate the database tables.

⚙️ How to Spin This Up Yourself
Want to test drive it? Follow these steps. (Warning: Running an EKS cluster on AWS costs real money. Don't forget to tear it down!)

Step 1: Build the Cloud

Bash
cd terraform
terraform init
terraform apply --auto-approve
Step 2: Connect your Terminal to the Cluster

Bash
aws eks update-kubeconfig --region ap-south-1 --name <your-cluster-name>
Step 3: Install the Watchdog (ArgoCD)

Bash
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
Step 4: Tell ArgoCD to Take Over

Bash
kubectl apply -f kubernetes/argocd/bootstrap-app.yaml
Grab that coffee. ArgoCD will now read the rest of the repository and deploy the microservices and monitoring tools automatically.

Step 5: Tear it Down (Save your wallet!)

Bash
cd terraform
terraform destroy --auto-approve
Created by Irfan Khan as part of my journey into Cloud & DevOps Engineering.
