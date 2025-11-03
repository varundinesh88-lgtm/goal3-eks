Goal 3 — 2048 EKS Deployment README

________________________________________

2048 Game Deployment on AWS EKS

🚀 **Project Overview**

This project demonstrates containerizing a web application (2048 game) with Docker, pushing it to Amazon ECR, and deploying it on Amazon EKS with Kubernetes. It also includes Horizontal Pod Autoscaling (HPA) to handle variable loads.

________________________________________

🏗️ **Architecture**

Docker (2048 App) --> ECR Repo --> EKS Cluster --> Deployment --> Service (LoadBalancer) --> Public Web Access
•	Docker: Containerizes the 2048 static web app.

•	ECR: Hosts the Docker image for Kubernetes deployment.

•	EKS: Managed Kubernetes cluster to run the application.

•	LoadBalancer Service: Exposes the app publicly.

•	HPA: Scales pods based on CPU usage.

________________________________________

📂 **Project Structure**

    goal3-eks/
    ├─ app/               # 2048 source code (HTML/CSS/JS)
    ├─ deployment.yaml    # Kubernetes Deployment manifest
    ├─ service.yaml       # Kubernetes Service manifest (LoadBalancer)
    ├─ hpa.yaml           # Horizontal Pod Autoscaler manifest
    └─ README.md
    
________________________________________

⚙️ **Deployment Steps**

1. Build and Push Docker Image
docker build -t 2048-eks:v1 ./app
docker tag 2048-eks:v1 <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/2048-eks:v1
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com
docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/2048-eks:v1
2. Create EKS Cluster
eksctl create cluster \
  --name 2048-cluster \
  --region us-east-1 \
  --zones us-east-1a \
  --nodegroup-name ng-1 \
  --node-type t3.small \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed
3. Deploy Application on Kubernetes
kubectl create namespace 2048-game
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
4. Access the App
kubectl get svc -n 2048-game
•	Open the EXTERNAL-IP in your browser.

•	The 2048 game should load successfully.

________________________________________

📈 **Scaling**

•	HPA is configured to maintain 50% CPU utilization, scaling pods between 2 and 5 replicas automatically.

________________________________________

🖼️ **Screenshots**
(Add screenshots of the 2048 game running and HPA metrics)

________________________________________

🏗️ **Project Architecture**

High-Level Overview

             ┌────────────────────────┐
             │        Developer       │
             │   (Code + Dockerfile)  │
             └──────────┬─────────────┘
                        │
                        ▼
              ┌────────────────────┐
              │  Amazon ECR (Repo) │
              │ Stores Docker Image│
              └──────────┬─────────┘
                        │
                        ▼
             ┌──────────────────────────────┐
             │  Amazon EKS (Kubernetes)     │
             │  • Deployment (Pods)         │
             │  • Service (LoadBalancer)    │
             │  • Namespace (2048-game)     │
             └──────────┬───────────────────┘
                        │
                        ▼
             ┌──────────────────────────────┐
             │ AWS Load Balancer (ELB)      │
             │ Exposes Public IP            │
             └──────────────────────────────┘
Flow Summary:

1.	Docker image built locally and pushed to ECR.
2.	Kubernetes manifests (deployment.yaml, service.yaml) deployed on EKS.
3.	LoadBalancer service exposes the 2048 web app to the internet.

________________________________________

⚙️ **Key Kubernetes Files**

deployment.yaml

Defines the number of replicas and container image:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: 2048-deployment
      namespace: 2048-game
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: 2048
      template:
        metadata:
          labels:
            app: 2048
        spec:
          containers:
            - name: 2048
              image: <your-account-id>.dkr.ecr.us-east-1.amazonaws.com/2048-eks:v1
              ports:
                - containerPort: 80

service.yaml

Exposes the app via LoadBalancer:

    apiVersion: v1
    kind: Service
    metadata:
      name: 2048-service
      namespace: 2048-game
    spec:
      type: LoadBalancer
      selector:
        app: 2048
      ports:
        - port: 80
          targetPort: 80
          
________________________________________

🔍 **Troubleshooting Notes**(Issue	- Cause	- Solution)

no basic auth credentials when pushing to ECR	- Not logged in to ECR	- Run `aws ecr get-login-password ...
LoadBalancer IP not showing	- EKS provisioning delay	- Wait 2–5 minutes or run kubectl get svc -n 2048-game -w
Site not loading	- Security group or region mismatch	- Ensure EKS cluster and ECR are in same region; verify inbound rules on the ELB

________________________________________

🧠 **Key Learnings**

•	Hands-on with Kubernetes deployments and services on AWS EKS
•	Experience with Docker image builds and ECR authentication
•	Understanding of LoadBalancer services and public IP exposure
•	End-to-end knowledge of containerized workloads on AWS

________________________________________
📂 **Repository Structure**

    2048-eks/
    ├── Dockerfile
    ├── deployment.yaml
    ├── service.yaml
    ├── README.md
    └── screenshots/
    
________________________________________

📸 **Screenshots**

1.	ECR repository page with uploaded image
2.	kubectl get pods -n 2048-game output
3.	LoadBalancer service with public IP
4.	2048 game opened in browser

________________________________________

📘 **References**

•	Kubernetes Official Docs
•	Amazon EKS Workshop
•	Docker Hub - Nginx Base Image
•	AWS ECR Docs

________________________________________
