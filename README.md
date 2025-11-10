# Project 2 - 2048 EKS Deployment

________________________________________

## 2048 Game Deployment on AWS EKS

2048 game opened in browser
<img width="1920" height="1080" alt="10" src="https://github.com/user-attachments/assets/950998b0-e98e-4038-965a-6a624a3f7f8b" />

________________________________________

### _**Project Overview**_

This project demonstrates containerizing a web application (2048 game) with Docker, pushing it to Amazon ECR, and deploying it on Amazon EKS with Kubernetes. It also includes Horizontal Pod Autoscaling (HPA) to handle variable loads.

________________________________________

### _**Architecture**_

Docker (2048 App) --> ECR Repo --> EKS Cluster --> Deployment --> Service (LoadBalancer) --> Public Web Access

•	Docker: Containerizes the 2048 static web app.

•	ECR: Hosts the Docker image for Kubernetes deployment.

•	EKS: Managed Kubernetes cluster to run the application.

•	LoadBalancer Service: Exposes the app publicly.

•	HPA: Scales pods based on CPU usage.

________________________________________

### _**Project Structure**_

    goal3-eks/
    ├─ app/               # 2048 source code (HTML/CSS/JS)
    ├─ deployment.yaml    # Kubernetes Deployment manifest
    ├─ service.yaml       # Kubernetes Service manifest (LoadBalancer)
    ├─ hpa.yaml           # Horizontal Pod Autoscaler manifest
    └─ README.md
    
________________________________________

### _**Deployment Steps**_

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

### _**Scaling**_

•	HPA is configured to maintain 50% CPU utilization, scaling pods between 2 and 5 replicas automatically.

________________________________________

### _**Project Architecture**_

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

### _**Key Kubernetes Files**_

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

### _**Troubleshooting Notes**_(Issue - Cause - Solution)

* No basic auth credentials when pushing to ECR - Not logged in to ECR - Run `aws ecr get-login-password and docker login.
* LoadBalancer IP not showing	- EKS provisioning delay - Waited some time, Run kubectl get svc -n 2048-game -w
* Site not loading - Security group or region mismatch - Ensure EKS cluster and ECR are in same region; verify inbound rules on the ELB

________________________________________

### _**Key Learnings**_

•	Hands-on with Kubernetes deployments and services on AWS EKS
•	Experience with Docker image builds and ECR authentication
•	Understanding of LoadBalancer services and public IP exposure
•	End-to-end knowledge of containerized workloads on AWS

________________________________________
### _**Repository Structure**_

    2048-eks/
    ├── Dockerfile
    ├── deployment.yaml
    ├── service.yaml
    ├── README.md
    └── screenshots/
    
________________________________________

### _**Screenshots**_

ECR repository
<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/1596a321-a2ec-47bb-b27b-990a292e335d" />

EKS Cluster
<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/34996625-b06b-4d5b-b5d8-21e3b6c742f4" />

kubectl get pods -n 2048-game output
<img width="1920" height="1080" alt="8" src="https://github.com/user-attachments/assets/a6402457-9b16-4ce7-911a-d203a363a0b3" />

LoadBalancer service with public IP
<img width="1920" height="1080" alt="9" src="https://github.com/user-attachments/assets/c68a4229-7506-4613-8af7-ffd1ea8a82a1" />

CloudWatch Container Insights
<img width="1920" height="1080" alt="11" src="https://github.com/user-attachments/assets/b451ad2c-4b3c-4916-874f-ab0e7a572515" />

________________________________________

### _**References**_

•	Kubernetes Official Docs
•	Amazon EKS Workshop
•	Docker Hub - Nginx Base Image
•	AWS ECR Docs

________________________________________
