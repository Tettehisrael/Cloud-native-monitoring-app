# Cloud-native-monitoring-app

## Project Overview
In the "Cloud Native Monitoring App" project, our objective is to craft a modern, scalable monitoring solution tailored for cloud-native applications. We commence by developing the core monitoring application in Python, which serves as the foundation of our system. To ensure effortless deployment and management, we containerize the Python application using Docker. The resulting Docker images are securely stored in an Amazon Elastic Container Registry (ECR), offering versioning capabilities and a centralized image repository. For orchestrating and scaling our application, we set up an Amazon Elastic Kubernetes Service (EKS) cluster. The ECR-hosted Docker images are pulled onto EKS nodes, facilitating the seamless deployment of our monitoring solution. To expose the application externally and ensure its accessibility over the internet, we employ Python scripts to define Kubernetes deployments and services. This integration of tools and technologies culminates in a robust cloud-native monitoring ecosystem, empowering us to monitor and manage cloud-native applications effectively.

## Project Steps
1. Install Dependencies from requirements.txt
2. Build the Python application.
3. Create a Dockerfile for the Python application.
4. Build the Docker image using the Dockerfile.
5. Create an Amazon Elastic Container Registry (ECR) repository using the Python boto3 module.
6. Confirm ECR Repository Creation and Push Docker Image
7. Create an Amazon Elastic Kubernetes Service (EKS) Cluster and Node Group
8. Create Kubernetes deployments and services using Python to make the app accessible over the internet.
9. Expose a Kubernetes Service using Port Forwarding

### Step 1
In this step, we'll install the necessary Python dependencies for the application from the requirements.txt file.
- Run the following command to install the required Python packages from the requirements.txt file: `pip3 install -r requirement.txt`

### Step 2 
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

### Step 3
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

### Step 4
Now that we have our Dockerfile ready, it's time to build the Docker image and run it as a container. Follow these commands to accomplish this:

To build the Docker image, open your terminal and navigate to the directory containing the Dockerfile. Execute the following command: `docker build -t my-python-app .`

Once the image is built successfully, you can run it as a container with the following command: `docker run -p 5000:5000 my-python-app` Replace "my-python-app" with the actual image ID if necessary. This command will start the container, making your Python application accessible on port 5000

### Step 5
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

### Step 6
After creating your Amazon Elastic Container Registry (ECR) repository using the Python boto3 module, you can verify its existence in the AWS Management Console. Follow these steps:

- Login to AWS Console: Access the AWS Management Console using your AWS account credentials.

- Confirm ECR Repo: Navigate to the ECR service in the AWS Console to confirm that your repository has been successfully created. You should see it listed in the repositories.

- Push Docker Image: To push your Docker image into the repository, you can obtain the push command from the AWS Console. AWS provides a specific command with authentication details that you can copy and run in your terminal. This command will upload your Docker image to the ECR repository.

### Step 7
In this step, we will create an Amazon Elastic Kubernetes Service (EKS) cluster from was console and configure a node group.

- Create EKS Cluster

- Create Node Group

- Connect to the Cluster Using kubectl: Run the following command to configure kubectl to use your new EKS cluster `aws eks --region <your-region> update-kubeconfig --name <your-cluster-name>`

### Step 8
Replace the ECR image URL with yours in the Define deployment section.
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

### Step 9
In this step, we'll expose a Kubernetes service using port forwarding, allowing you to access your application locally. Follow these instructions:
- Use the following kubectl command to set up port forwarding to your Kubernetes service: `kubectl port-forward svc/<service-name> <local-port>:<service-port>`
- Once port forwarding is established, you can access your application by opening a web browser or making API requests to `http://localhost:<local-port>`.




 


