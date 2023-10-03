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
In this step, we build the core Python application that forms the basis of our monitoring solution.

```
import psutil
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    cpu_metric = psutil.cpu_percent()
    mem_metric = psutil.virtual_memory().percent
    Message = None
    if cpu_metric > 80 or mem_metric > 80:
        Message = "High CPU or Memory Detected, scale up!!!"
    return render_template("index.html", cpu_metric=cpu_metric, mem_metric=mem_metric, message=Message)

if __name__=='__main__':
    app.run(debug=True, host = '0.0.0.0')
```
To execute the Python application, run the following command in your terminal: `python3 app.py`

### Step 2
To containerize our Python application, we create a Dockerfile that specifies how the application should be packaged into a Docker image.
```
# Use the official Python image as the base image
FROM python:3.9-buster

#Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt

# Install the required Python packages
RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is running
CMD ["flask", "run"]
```

### Step 3
With the Dockerfile in place, we proceed to build the Docker image of our Python application. Execute the following command in your terminal to build the Docker image: `docker build -t my-python-app .`

### Step 4
We create an Amazon ECR repository to securely store our Docker images and ensure versioning and accessibility.
```
import boto3

ecr_client = boto3.client('ecr', region_name='us-east-1')

repository_name = "my_monitoring_app_image"
response = ecr_client.create_repository(repositoryName=repository_name)

repository_uri = response ['repository']['repositoryUri']
print(repository_uri)
```
Run the following Python script to create an ECR repository: `python3 ecr.py`

### Step 5
In this step, we set up an Amazon EKS cluster with nodes to orchestrate and manage our application.
Replace ECR image url with yours in the deployment section.
```
#create deployment and service
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="690688793859.dkr.ecr.us-east-1.amazonaws.com/my_monitoring_app_image",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```
Execute the following Python script to set up an EKS cluster: `python3 eks.py`




 


