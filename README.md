# AI/ML Internal Developer Platform

A production-grade Internal Developer Platform that enables data scientists to self-service GPU workloads while platform teams maintain cost visibility and governance—addressing the operational complexity organisations face when scaling ML infrastructure.

## Overview

Organisations adopting AI/ML workloads quickly discover that GPU resources are expensive, allocation is chaotic, and data scientists waste time on infrastructure instead of model development. This platform solves that by providing a self-service portal where teams can provision GPU training jobs, deploy models, and track costs—all while platform engineering maintains governance through automated controls.

The architecture uses a hybrid EKS/ECS approach: Kubernetes handles the complex orchestration of ML tools (Backstage, MLflow, ArgoCD), while the GPU cost tracking service runs on ECS Fargate. This separation allows cost-optimised compute for simple services while preserving Kubernetes flexibility for platform workloads. GitOps via ArgoCD ensures all deployments are version-controlled and self-healing.

Backstage templates abstract away infrastructure complexity—developers select a framework, specify GPU count, and set runtime limits. The platform checks budget constraints, allocates resources, registers models in MLflow, and creates ArgoCD applications automatically. This demonstrates how platform engineering reduces cognitive load while enforcing organisational policies.

## Architecture

![Cloud Architecture](screenshots/architecture.png)

The platform spans two compute services within a shared VPC. EKS runs the developer-facing tools: Backstage serves as the unified portal, MLflow handles model registry and experiment tracking, and ArgoCD manages GitOps deployments. A custom GPU Cost Tracker service runs independently on ECS Fargate, providing real-time spend visibility and resource allocation APIs.

Request flow starts at Backstage: a developer uses a software template to request GPU resources. The template triggers API calls to the Cost Tracker to verify budget availability, then allocates GPUs and registers the model in MLflow. ArgoCD detects the new application manifest and deploys the workload to the cluster. The Cost Tracker continuously aggregates spend data, surfacing it through a dashboard accessible to both developers and finance.

Terraform provisions the complete infrastructure—VPC with public/private subnets, EKS cluster with managed node groups, ECS cluster with Fargate tasks, ALB for external access, ECR for container images, and all supporting IAM roles and security groups. The Helm provider deploys platform services directly from Terraform, creating a single deployment pipeline from infrastructure to application.

## Tech Stack

**Infrastructure**: AWS EKS, ECS Fargate, VPC, ALB, ECR, Terraform  
**Platform Services**: Backstage, MLflow, ArgoCD  
**Deployment**: Helm, GitOps, Kubernetes  
**Custom Service**: Node.js, Express, Chart.js  
**Observability**: CloudWatch Container Insights

## Key Decisions

- **Hybrid EKS/ECS architecture**: Kubernetes excels at orchestrating complex, stateful platform services, but running simple API services on EKS adds unnecessary overhead. The Cost Tracker runs on Fargate at ~70% lower cost than equivalent EKS capacity, demonstrating cost-conscious architecture decisions.

- **Backstage as the developer portal**: Rather than exposing raw Kubernetes, MLflow, and ArgoCD interfaces, Backstage provides a unified abstraction layer. Software templates encode organisational standards—GPU limits, budget checks, naming conventions—making compliance automatic rather than manual.

- **Budget enforcement at allocation time**: The Cost Tracker doesn't just report spend after the fact; it integrates with Backstage templates to approve or reject GPU requests based on remaining budget. This prevents runaway costs before resources are provisioned.

- **GitOps for ML deployments**: Model deployments flow through ArgoCD rather than direct kubectl applies. This creates an audit trail of what was deployed, enables rollbacks, and ensures the cluster state matches the Git repository—critical for regulated ML environments.

## Screenshots

![ArgoCD dashboard](screenshots/argocd-dashboard.png)

![Cost Tracker interface](screenshots/cost-tracker.png)

![Backstage portal](screenshots/backstage-portal.png)

![MLflow registry](screenshots/mlflow-registry.png)

## Author

**Noah Frost**

- Website: [noahfrost.co.uk](https://noahfrost.co.uk)
- GitHub: [github.com/nfroze](https://github.com/nfroze)
- LinkedIn: [linkedin.com/in/nfroze](https://linkedin.com/in/nfroze)