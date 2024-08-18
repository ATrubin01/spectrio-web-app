# Spectrio Web Application Deployment
This project demonstrates the deployment of a simple web application to an Amazon EKS (Elastic Kubernetes Service) cluster using CI/CD pipelines via GitHub Actions. The application is containerized using Docker, pushed to Amazon ECR (Elastic Container Registry), and deployed to EKS, leveraging Kubernetes manifests including Deployment, Service, and Ingress configurations.

## Prerequisites
  - AWS CLI: Installed and configured with the necessary permissions.
  - kubectl: Installed for interacting with the Kubernetes cluster.
  - GitHub Actions: Configured in your GitHub repository.
  - Amazon EKS: An existing EKS cluster.
  - Amazon ECR: An ECR repository to push Docker images.
  - Ingress Controller: Nginx Ingress Controller installed on your EKS cluster.
## Setup and Deployment
### 1. Dockerize the Web Application
The application is built using Docker. The Dockerfile specifies the steps to containerize the application.

### 2. Configure CI/CD Pipeline
The CI/CD pipeline is defined using GitHub Actions in .github/workflows/web-app.yml. It performs the following steps:

  - Checkout Code: Clones the repository.
  - Login to AWS: Authenticates with AWS using OIDC and GitHub Secrets.
  - Build and Push Docker Image: Builds the Docker image from the Dockerfile and pushes it to Amazon ECR.
  - Update Kubernetes Configuration: Updates the kubeconfig to access the EKS cluster.
  - Deploy to Kubernetes: Applies the Kubernetes manifests to deploy the application, service, and ingress.
### 3. Kubernetes Manifests
  - Deployment (deployment.yaml): Defines the desired state for the web application, ensuring that 2 replicas are running.
  - Service (service.yaml): Exposes the application within the Kubernetes cluster and creates a LoadBalancer to allow external access.
  - Ingress (ingress.yaml): Configures the Ingress resource to route traffic from a specified domain to the application service.
### 4. Accessing the Application
After deployment, the application can be accessed using the configured domain. The Ingress controller routes the traffic to the service, which forwards it to the application pods.

### 5. Ensuring the Application Runs on Different Nodes
With 2 replicas specified in the Deployment manifest, Kubernetes schedules the pods across different nodes in the EKS cluster, ensuring high availability.

### 6. Scaling the Application
You can scale the application by modifying the replicas field in the deployment.yaml and reapplying the manifest. The Cluster Autoscaler, if configured, will automatically adjust the number of nodes based on the load.

## Key Files
  - Dockerfile: Instructions to build the Docker image.
  - deployment.yaml: Defines the deployment for the web application.
  - service.yaml: Exposes the application via a Kubernetes Service.
  - ingress.yaml: Configures Ingress for routing external traffic.
## Notes
  - Ensure that the appropriate IAM roles and policies are attached to the EKS cluster and GitHub Actions for successful deployment.
  - The application’s external access is managed by the Ingress controller, which requires a properly configured domain name and DNS settings.
## Troubleshooting
  - 502 Bad Gateway: Check if the Ingress is correctly configured and the service is correctly exposed.
  - Load Balancer Issues: Ensure that the public subnets are correctly tagged with kubernetes.io/role/elb=1 and that the Ingress Controller has an external IP assigned.
<img width="514" alt="Screenshot 2024-08-18 at 1 12 37 PM" src="https://github.com/user-attachments/assets/ce992516-7cb0-437a-963e-ae99f6c58dd3">



# Podinfo

## EKS Cluster Autoscaler and Deployment
## Overview
This section of the project demonstrates the setup and functionality of the Cluster Autoscaler within an Amazon EKS cluster. It includes deploying a web application (podinfo), configuring node selectors and tolerations, and testing the autoscaler by scaling the deployment.

## Deployment of Web Application
### 1. Deploying the podinfo Application in Node Group 02
I deployed the podinfo application to a specific node group (Node Group 02) in the EKS cluster. This setup is used to test the Cluster Autoscaler's ability to manage node scaling based on resource requests.

Deployment Manifest:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podinfo-node-group-02
  labels:
    app: podinfo
spec:
  replicas: 1 # Initial replica count
  selector:
    matchLabels:
      app: podinfo
  template:
    metadata:
      labels:
        app: podinfo
    spec:
      nodeSelector:
        eks.amazonaws.com/nodegroup: spectrio-project-node-group-02-dev
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "podinfo"
        effect: "NoSchedule"
      containers:
      - name: podinfo
        image: stefanprodan/podinfo
        ports:
        - containerPort: 9898
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```
Apply the Manifest:

```
kubectl apply -f podinfo-node-group-02-deployment.yaml
```
### 2. Scaling the Deployment
To test the Cluster Autoscaler, I scaled the deployment to a higher number of replicas. This action should trigger the autoscaler to add more nodes to the cluster.

Scale Up Command:

```
kubectl scale deployment podinfo-node-group-02 --replicas=40
```
### 3. Observing Cluster Autoscaler
Monitor the Cluster Autoscaler to ensure it adds nodes to Node Group 02 based on the increased replica count. You can check the Cluster Autoscaler logs and EKS cluster status for confirmation.

### 4. Scaling Down the Deployment
After verifying that the autoscaler added nodes, scale down the deployment to test the autoscaler's ability to remove excess nodes.

Scale Down Command:

```
kubectl scale deployment podinfo-node-group-02 --replicas=1
```
### Verify Node Scaling:

Check that the Cluster Autoscaler removes the extra nodes from Node Group 02 as expected.

## Challenges
Pods Not Scheduling: Ensure that node selectors and tolerations are correctly configured. Check the node labels to match the nodeSelector values.
Cluster Autoscaler Logs: Check the logs for any errors or warnings related to autoscaling actions.

