# Cloud-native-monitoring-app

## Project Overview
In the "Cloud Native Monitoring App" project, our objective is to craft a modern, scalable monitoring solution tailored for cloud-native applications. We commence by developing the core monitoring application in Python, which serves as the foundation of our system. To ensure effortless deployment and management, we containerize the Python application using Docker. The resulting Docker images are securely stored in an Amazon Elastic Container Registry (ECR), offering versioning capabilities and a centralized image repository. For orchestrating and scaling our application, we set up an Amazon Elastic Kubernetes Service (EKS) cluster. The ECR-hosted Docker images are pulled onto EKS nodes, facilitating the seamless deployment of our monitoring solution. To expose the application externally and ensure its accessibility over the internet, we employ Python scripts to define Kubernetes deployments and services. This integration of tools and technologies culminates in a robust cloud-native monitoring ecosystem, empowering us to monitor and manage cloud-native applications effectively.

## Project Steps
1. Build the Python application.
2. Create a Dockerfile for the Python application.
3. Build the Docker image using the Dockerfile.
4. Create an Amazon Elastic Container Registry (ECR) repository using the Python boto3 module.
5. Set up an Amazon Elastic Kubernetes Service (EKS) cluster with nodes.
6. Deploy the Python application on Kubernetes.
7. Create Kubernetes deployments and services using Python to make the app accessible over the internet.

### Step 1 


