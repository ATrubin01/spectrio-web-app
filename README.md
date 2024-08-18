Spectrio Web Application Deployment
This project demonstrates the deployment of a simple web application to an Amazon EKS (Elastic Kubernetes Service) cluster using CI/CD pipelines via GitHub Actions. The application is containerized using Docker, pushed to Amazon ECR (Elastic Container Registry), and deployed to EKS, leveraging Kubernetes manifests including Deployment, Service, and Ingress configurations.

Project Structure
bash
Copy code
├── Dockerfile                   # Dockerfile for building the web application image
├── deployment.yaml              # Kubernetes Deployment manifest
├── service.yaml                 # Kubernetes Service manifest
├── ingress.yaml                 # Kubernetes Ingress manifest
├── .github/workflows
│   └── ci-cd-pipeline.yml       # GitHub Actions workflow for CI/CD pipeline
└── README.md                    # Project README file
Prerequisites
AWS CLI: Installed and configured with the necessary permissions.
kubectl: Installed for interacting with the Kubernetes cluster.
GitHub Actions: Configured in your GitHub repository.
Amazon EKS: An existing EKS cluster.
Amazon ECR: An ECR repository to push Docker images.
Ingress Controller: Nginx Ingress Controller installed on your EKS cluster.
Setup and Deployment
1. Dockerize the Web Application
The application is built using Docker. The Dockerfile specifies the steps to containerize the application.

2. Configure CI/CD Pipeline
The CI/CD pipeline is defined using GitHub Actions in .github/workflows/ci-cd-pipeline.yml. It performs the following steps:

Checkout Code: Clones the repository.
Login to AWS: Authenticates with AWS using OIDC and GitHub Secrets.
Build and Push Docker Image: Builds the Docker image from the Dockerfile and pushes it to Amazon ECR.
Update Kubernetes Configuration: Updates the kubeconfig to access the EKS cluster.
Deploy to Kubernetes: Applies the Kubernetes manifests to deploy the application, service, and ingress.
3. Kubernetes Manifests
Deployment (deployment.yaml): Defines the desired state for the web application, ensuring that 2 replicas are running.
Service (service.yaml): Exposes the application within the Kubernetes cluster and creates a LoadBalancer to allow external access.
Ingress (ingress.yaml): Configures the Ingress resource to route traffic from a specified domain to the application service.
4. Accessing the Application
After deployment, the application can be accessed using the configured domain. The Ingress controller routes the traffic to the service, which forwards it to the application pods.

5. Ensuring the Application Runs on Different Nodes
With 2 replicas specified in the Deployment manifest, Kubernetes schedules the pods across different nodes in the EKS cluster, ensuring high availability.

6. Scaling the Application
You can scale the application by modifying the replicas field in the deployment.yaml and reapplying the manifest. The Cluster Autoscaler, if configured, will automatically adjust the number of nodes based on the load.

Key Files
Dockerfile: Instructions to build the Docker image.
deployment.yaml: Defines the deployment for the web application.
service.yaml: Exposes the application via a Kubernetes Service.
ingress.yaml: Configures Ingress for routing external traffic.
Notes
Ensure that the appropriate IAM roles and policies are attached to the EKS cluster and GitHub Actions for successful deployment.
The application’s external access is managed by the Ingress controller, which requires a properly configured domain name and DNS settings.
Troubleshooting
502 Bad Gateway: Check if the Ingress is correctly configured and the service is correctly exposed.
Load Balancer Issues: Ensure that the public subnets are correctly tagged with kubernetes.io/role/elb=1 and that the Ingress Controller has an external IP assigned.
Conclusion
This project showcases the deployment of a simple web application to an EKS cluster, utilizing GitHub Actions for continuous integration and deployment. The use of Kubernetes manifests for defining infrastructure and application configurations ensures a scalable and reliable deployment process.