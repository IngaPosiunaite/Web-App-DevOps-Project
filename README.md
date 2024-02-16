## Project Status 
"The last two tasks of the project are still in development **02/16/2024**

# Web-App-DevOps-Project

Welcome to the Web App DevOps Project repo! This application allows you to efficiently manage and track orders for a potential business. It provides an intuitive user interface for viewing existing orders and adding new ones.

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
- [Technology Stack](#technology-stack)
- [Contributors](#contributors)
- [License](#license)
- [Containerization with Docker](#containerization-with-Docker)
- [Terraform Project and Modules](#terraform-project-and-modules)
- [Kubernetes Deployment with AKS](#kubernetes-deployment-with-aks)
- [CI/CD Pipeline with Azure DevOps](#ci-cd-pipeline-with-azure-devops)
- [AKS Cluster Monitoring](#aks-cluster-monitoring)
- [AKS Integration with Azure Key Vault for Secret Management](#aks-integration-with-azure-key-vault-for-secret-management)



## Features

- **Order List:** View a comprehensive list of orders including details like date UUID, user ID, card number, store code, product code, product quantity, order date, and shipping date.
  
![Screenshot 2023-08-31 at 15 48 48](https://github.com/maya-a-iuga/Web-App-DevOps-Project/assets/104773240/3a3bae88-9224-4755-bf62-567beb7bf692)

- **Pagination:** Easily navigate through multiple pages of orders using the built-in pagination feature.
  
![Screenshot 2023-08-31 at 15 49 08](https://github.com/maya-a-iuga/Web-App-DevOps-Project/assets/104773240/d92a045d-b568-4695-b2b9-986874b4ed5a)

- **Add New Order:** Fill out a user-friendly form to add new orders to the system with necessary information.
  
![Screenshot 2023-08-31 at 15 49 26](https://github.com/maya-a-iuga/Web-App-DevOps-Project/assets/104773240/83236d79-6212-4fc3-afa3-3cee88354b1a)

- **Data Validation:** Ensure data accuracy and completeness with required fields, date restrictions, and card number validation.

## Getting Started

### Prerequisites

For the application to succesfully run, you need to install the following packages:

- flask (version 2.2.2)
- pyodbc (version 4.0.39)
- SQLAlchemy (version 2.0.21)
- werkzeug (version 2.2.3)

### Usage

To run the application, you simply need to run the `app.py` script in this repository. Once the application starts you should be able to access it locally at `http://127.0.0.1:5000`. Here you will be meet with the following two pages:

1. **Order List Page:** Navigate to the "Order List" page to view all existing orders. Use the pagination controls to navigate between pages.

2. **Add New Order Page:** Click on the "Add New Order" tab to access the order form. Complete all required fields and ensure that your entries meet the specified criteria.

## Technology Stack

- **Backend:** Flask is used to build the backend of the application, handling routing, data processing, and interactions with the database.

- **Frontend:** The user interface is designed using HTML, CSS, and JavaScript to ensure a smooth and intuitive user experience.

- **Database:** The application employs an Azure SQL Database as its database system to store order-related data.

## Contributors 

- [Maya Iuga]([https://github.com/yourusername](https://github.com/maya-a-iuga))

## License

This project is licensed under the MIT License. For more details, refer to the [LICENSE](LICENSE) file.

## Containerization with Docker

- **Containerization Process**: This section documents the steps taken to containerize the application using Docker. It includes the Dockerfile with explanations for each instruction and details on building and running Docker containers locally.

- **Docker Commands**: Documented commands include building the Docker image, running containers, tagging images, and pushing to Docker Hub. Examples and explanations are provided for each command.

- **Image Information**: Essential information about the Docker image, such as its name, tags, and usage instructions, is documented.

## Terraform Project and Modules

This Terraform module is designed to provision the necessary Azure Networking Services for an Azure Kubernetes Service (AKS) cluster. This project utilizes Terraform to provision an Azure Kubernetes Service (AKS) cluster along with the necessary networking infrastructure. The Infrastructure as Code (IaC) approach ensures a reproducible and scalable deployment.

The project is organized into two Terraform modules:

- **networking-module**: Responsible for provisioning Azure Networking Services.
- **aks-cluster-module**: Focuses on provisioning the AKS cluster.

The main.tf file in the project directory serves as the main configuration file. It orchestrates the deployment of the AKS cluster and its associated networking resources.

### Azure Provider Setup

The Azure provider block authenticates Terraform to Azure using service principal credentials. These credentials are stored as environment variables to enhance security.

```
provider "azurerm" {
  features {}

  client_id       = var.client_id
  client_secret   = var.client_secret
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}
```
### Networking Module Integration

The networking module is integrated into the main configuration file to ensure the creation of essential networking resources for the AKS cluster.

```
module "networking" {
  source              = "./networking-module"
  resource_group_name = "networking-resource-group"
  location            = "UK South"
  vnet_address_space  = ["10.0.0.0/16"]
}
```
### Cluster Module Integration

Similarly, the cluster module is integrated to define and provision the AKS cluster within the networking infrastructure.

```
module "aks_cluster" {
  source                   = "./aks-cluster-module"
  cluster_name             = "terraform-aks-cluster"
  location                 = "UK South"
  dns_prefix               = "myaks-project"
  kubernetes_version       = "1.26.6"
  service_principal_client_id     = var.service_principal_client_id
  service_principal_secret         = var.service_principal_secret
  resource_group_name     = module.networking.networking_resource_group_name
  vnet_id                 = module.networking.vnet_id
  control_plane_subnet_id = module.networking.control_plane_subnet_id
  worker_node_subnet_id   = module.networking.worker_node_subnet_id
  aks_nsg_id              = module.networking.aks_nsg_id
}
```

### Input Variables

The following input variables are defined in the variables.tf file to customize various aspects of the AKS cluster:

- **client_id**: Azure service principal client ID
- **client_secret**: Azure service principal client secret
- **subscription_id**: Azure subscription ID
- **tenant_id**: Azure tenant ID
- **service_principal_client_id**: Client ID for the AKS service principal
- **service_principal_secret**: Client secret for the AKS service principal

### Usage

1. Initialize the Terraform project:

```
terraform init
```
2. Apply the Terraform configuration:

```
terraform apply
```
3. Retrieve the kubeconfig file:

```
az aks get-credentials --resource-group networking-resource-group --name terraform-aks-cluster
```
4. Verify the AKS cluster connectivity:

```
kubectl get nodes
```

## Kubernetes Deployment with AKS

### Deployment and Service Manifests

We have defined the Deployment and Service manifests to deploy our application on Kubernetes within our AKS cluster.

### Deployment Manifest:

In the Deployment manifest (application-manifest.yaml), we specify the desired state for our application pods, including the number of replicas, container image, ports, and rolling update strategy.

```
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask-app-container
          image: ingeera/webapp
          ports:
            - containerPort: 5000
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

### Key concepts:

- **Replicas**: We have configured two replicas to ensure high availability and scalability.
- **Container Image**: We reference our application container image hosted on Docker Hub.
- **Ports**: We expose port 5000 within the container for internal communication.
- **Rolling Update Strategy**: We use a RollingUpdate strategy to update our application seamlessly without downtime.

## Service Manifest:

In the Service manifest within the same application-manifest.yaml file, we define how other pods within the cluster can communicate with our application.

### Key configurations:

- **Selector**: We match the labels specified in the Deployment manifest to route traffic to the correct pods.
- **Ports**: We expose port 80 internally, which forwards traffic to port 5000 on the pods.

## Deployment Strategy

We have chosen the RollingUpdate deployment strategy for our application. This strategy gradually replaces existing pods with new ones, ensuring that the application remains available during updates. This aligns with our application's requirements for minimal downtime and uninterrupted service for users.

## Testing and Validation

After deployment, we conducted thorough testing and validation to ensure the functionality and reliability of our application within the AKS cluster. 

## Application Distribution Plan

To distribute the application to other internal users within our organization without relying on port forwarding, we plan to:

1. **Create an Ingress Controller**: Set up an Ingress controller to route external traffic to our application securely.
2. **Configure DNS**: Assign a domain name to the Ingress controller to provide a user-friendly URL for accessing the application.
3. **Implement RBAC**: Use Role-Based Access Control (RBAC) to manage access to the application based on user roles and permissions.
4. **Secure External Access**: Implement SSL termination and other security measures to secure external access to the application.


## CI/CD Pipeline with Azure DevOps

### Introduction
This document outlines the CI/CD pipeline setup using Azure DevOps for our project. The pipeline automates the build and deployment process, ensuring seamless integration and delivery of our application.

## Configuration Details
### Source Repository
- GitHub is configured as the source control system.
- The repository chosen for the pipeline is [Web-App-DevOps-Project].

### Build Pipeline
- A Starter Pipeline template is used as the foundation.
- Docker tasks are incorporated to build and push the Docker image to Docker Hub.
  - Personal access token on Docker Hub is used for authentication.
  - Docker image is pushed to Docker Hub upon successful build.
- Pipeline triggers on each push to the main branch of the repository.

### Release Pipeline
- The Deploy to Kubernetes task is integrated into the release pipeline.
- AKS service connection is established to facilitate deployment to the AKS cluster.
- The deployment manifest available in the repository is utilized for deployment.
- Automatic deployment to the AKS cluster is triggered upon successful build.

### Integration with Docker Hub and AKS
- Service connections are set up between Azure DevOps and Docker Hub, and Azure DevOps and AKS.
- Docker Hub service connection utilizes a personal access token for authentication.
- AKS service connection enables secure deployment to the AKS cluster.

## Validation Steps
- After configuring the CI/CD pipeline, validation steps were performed to ensure functionality.
- Monitoring of pod status within the AKS cluster confirmed successful creation.
- Port forwarding using `kubectl` was initiated to access the application securely.
- The functionality of the application was thoroughly tested to ensure correct operation within the AKS cluster.
- Any encountered issues were documented for further analysis and resolution.

## Conclusion
The CI/CD pipeline in Azure DevOps streamlines the build and deployment process, enabling efficient delivery of our application to the AKS cluster. Continuous testing and validation ensure reliability and functionality of the deployed application.

