```mermaid
graph LR
    %% Стилизация блоков
    classDef dev fill:#4B5563,stroke:#fff,stroke-width:2px,color:#fff;
    classDef github fill:#24292e,stroke:#fff,stroke-width:2px,color:#fff;
    classDef aws fill:#FF9900,stroke:#232F3E,stroke-width:2px,color:#232F3E;
    classDef k8s fill:#326CE5,stroke:#fff,stroke-width:2px,color:#fff;
    classDef users fill:#10B981,stroke:#fff,stroke-width:2px,color:#fff;

    %% Разработчик и Git
    Dev[Developer]:::dev -->|Triggers via git push| GH[GitHub Repository]:::github

    %% Секция CI/CD (GitHub Actions)
    subgraph CI_CD [GitHub Actions CI/CD]
        GH -->|Runs Workflow| GHA[GitHub Actions Runner]:::github
        GHA -->|Requests Token| OIDC[GitHub OIDC]:::github
        OIDC -->|Assumes| Role[AWS IAM Role]:::aws
        GHA -->|Executes| Docker[Build Docker Image]:::github
    end

    %% Секция AWS Registry & Deploy
    Docker -->|Pushes Image to| ECR[Amazon ECR]:::aws
    Role -->|Applies Manifests via| Helm[Helm Upgrade]:::k8s

    %% Секция Kubernetes & Трафик
    subgraph EKS_Cluster [Amazon EKS Cluster]
        Helm -->|Creates / Updates| Deploy[Deployment / Pods]:::k8s
        Ingress[Kubernetes Ingress]:::k8s -->|Exposes| Service[Kubernetes Service]:::k8s
        Service -->|Targets| Deploy
    end

    %% Внешний трафик (Вход в кластер)
    Users[Users / Clients]:::users -->|Sends HTTP Requests| ALB[AWS Application Load Balancer]:::aws
    ALB -->|Routes to| Ingress

    %% Связь ECR и Deployment
    Deploy -.->|Pulls Active Image from| ECR
```


# Go API on EKS with GitHub Actions CI/CD

## Overview

This project demonstrates a complete CI/CD pipeline for deploying a Go application to Amazon EKS.

The pipeline automatically:

* Builds a Docker image
* Pushes the image to Amazon ECR
* Deploys the application to EKS using Helm
* Exposes the application through AWS Load Balancer Controller and ALB Ingress

Infrastructure provisioning is managed separately in the Terraform EKS project.

---

## Related Infrastructure Repository

The Kubernetes cluster and AWS infrastructure are provisioned using Terraform:

**Terraform EKS Infrastructure Repository**

* VPC
* EKS Cluster
* Managed Node Groups
* OIDC Provider
* IRSA
* AWS Load Balancer Controller
* S3 Remote State

This repository assumes that infrastructure already exists. Check https://github.com/whosthefunkyy/terraform-to-eks

---

## Tech Stack

* Go
* Docker
* Kubernetes
* Helm
* Amazon EKS
* Amazon ECR
* AWS ALB Ingress Controller
* GitHub Actions
* GitHub OIDC

---

## CI/CD Flow
GitHub Push → GitHub Actions → Build Docker Image → Push Image to Amazon ECR → Helm Upgrade / Install → Amazon EKS → AWS ALB Ingress → Application Available Online

## Repository Structure
<img width="633" height="667" alt="Снимок экрана 2026-06-10 в 20 47 19" src="https://github.com/user-attachments/assets/c9894b92-cf49-403c-941b-865a7b1b4a79" />


## Kubernetes Components

This project deploys:

* Deployment
* Service (ClusterIP)
* Ingress (ALB)
* Horizontal Pod Autoscaler
* ServiceAccount
* ConfigMap
* ServiceMonitor (optional)

---

## Security

Authentication between GitHub Actions and AWS uses GitHub OIDC.

No long-lived AWS Access Keys are stored in GitHub Secrets.

GitHub assumes an IAM Role using OpenID Connect and receives temporary AWS credentials during workflow execution.

---

## Deployment

Every push to the main branch triggers:

1. Docker image build
2. Push to Amazon ECR
3. Helm deployment to EKS
4. Rollout verification

---

## Result

After deployment the application becomes accessible through an AWS Application Load Balancer created automatically by Kubernetes Ingress.
