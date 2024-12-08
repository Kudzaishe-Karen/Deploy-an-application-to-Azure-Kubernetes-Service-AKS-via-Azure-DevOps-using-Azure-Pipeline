# Deploy-to-AKS

Repository for showcasing the deployment from Azure DevOps Pipelines to AKS.

# Deployment to AKS from Azure DevOps Pipelines

This document outlines the process of deploying applications to Azure Kubernetes Service (AKS) using Azure DevOps Pipelines. 

## Prerequisites

- **Azure Subscription**: An active Azure account with the necessary permissions to manage AKS and Azure DevOps resources.
- **Azure DevOps Project**: Configured with the repository containing your application code.
- **AKS Cluster**: An AKS cluster where your application will be deployed.
- **Service Connection**: In Azure DevOps, set up a service connection to your Azure subscription for automation.

## Setup

### 1. Azure DevOps Configuration

- **Repository**: Ensure your application code is in a Git repository accessible by Azure DevOps.
- **Service Connection**:
  - Navigate to **Project Settings** > **Service Connections**.
  - Add an **Azure Resource Manager** service connection for your Azure subscription.

### 2. Kubernetes Configuration

- **kubectl**: Ensure `kubectl` is installed and configured to communicate with your AKS cluster.
- **Azure CLI**: Installed and logged into your Azure account with `az login`.

### 3. Pipeline Setup

Here's an example YAML for an Azure DevOps Pipeline:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  imageRepository: 'mycontainerregistry.azurecr.io/myapp'
  containerRegistry: 'myContainerRegistryConnection'
  dockerfilePath: '**/Dockerfile'
  tag: '$(Build.BuildId)'
  imagePullSecret: 'acr-auth'

steps:
- task: Docker@2
  displayName: Build and Push Docker Image
  inputs:
    command: buildAndPush
    repository: $(imageRepository)
    dockerfile: $(dockerfilePath)
    containerRegistry: $(containerRegistry)
    tags: |
      $(tag)

- task: KubernetesManifest@0
  displayName: Deploy to AKS
  inputs:
    action: 'deploy'
    namespace: 'default'
    manifests: |
      manifests/deployment.yml
      manifests/service.yml
    imagePullSecrets: $(imagePullSecret)
    containers: |
      $(imageRepository):$(tag)

4. Kubernetes Manifests
Create or update your Kubernetes manifests in a manifests directory:

deployment.yml:

yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: mycontainerregistry.azurecr.io/myapp:$(Build.BuildId)
        ports:
        - containerPort: 80

service.yml:

yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: myapp

Execution
Commit & Push: Push changes to your repository's main branch to trigger the pipeline.
Monitor: Check the Azure DevOps pipeline execution for logs and status.

Troubleshooting
Pipeline Failures: Review logs in Azure DevOps for specific errors.
Deployment Issues: Use kubectl commands to debug deployments on the AKS cluster.

Additional Notes
Security: Always manage secrets securely, using Azure Key Vault or Kubernetes secrets.
Scaling: Consider using Helm for more complex deployments or if you need to manage multiple environments.
CI/CD Best Practices: Implement proper branching strategies, automated testing, and continuous integration before deployment steps.

For more detailed guidance or specific configurations, refer to the Azure DevOps and AKS documentation.

This `README.md` provides an overview and steps for setting up CI/CD from Azure DevOps to AKS, which should give you a good starting point for your project deployment. Remember to adjust paths, names, and settings according to your specific setup. 
