# Deploying Your First Application Image To Kubernetes

This hands-on tutorial provides a step-by-step guide to preparing an application image, setting up the necessary infrastructure, and deploying the image to a Kubernetes cluster. It is written for professional software developers who are new to Kubernetes and containerization.

## Prerequisites

- **Docker**: Ensure you have Docker installed on your machine. If not, please click on this [link](https://www.docker.com/get-started/) to download it and follow the instructions for your operating system and hardware to install it.
- **Access to a Kubernetes cluster**: You'll need access to a running Kubernetes cluster. While it's possible to use a local cluster like Minikube or Docker Desktop, since we are currently migrating our microservices from an on-premises data center to cloud infrastructure, in a real scenario, you will most likely use a cloud-based cluster like Google Kubernetes Engine, Amazon EKS, or Azure Kubernetes Service.
- **kubectl**: This is a Kubernetes command-line tool used for managing clusters. You will need it to deploy your application image to a cluster. [Download and install it](https://kubernetes.io/docs/tasks/tools) based on your operating system.

## Step 1 - Set Up the Project Directory Structure

First, create a directory structure to organize your project files:

```
  mkdir -p quickstart_docker/application
  mkdir -p quickstart_docker/docker/application
```

Your project directory (`quickstart_docker`) should now look like this:

```
  quickstart_docker/      # Root directory for the project
  ├── application/        # Holds the application code
  └── docker/             # Holds Docker-related files
      └── application/    # Holds the Dockerfile for the application
```

## Step 2 - Create a Sample Application

Next, create a simple Python application that will be containerized and deployed. Inside the `quickstart_docker/application` directory, create a file named `application.py` with the following piece of code:

```
  import http.server
  import socketserver

  PORT = 8000

  Handler = http.server.SimpleHTTPRequestHandler

  httpd = socketserver.TCPServer(("", PORT), Handler)

  print("serving at port", PORT)
  httpd.serve_forever()
```

This script sets up a simple HTTP server that listens on port 8000.

## Step 3 - Create a Dockerfile

You may have noticed that we didn't mention in our Prerequisites section that you need to install Python and its dependencies based on your operating system. That's because we will be using an official Python image (version 3.5) from Docker Hub.

To create a container for the application, you must first set up the environment for your Python application. When using a Docker container, this is done with a file named `Dockerfile`. Create this file in the  `quickstart_docker/docker/application` directory and add the following lines to it:

```
# Use base image from the registry
FROM python:3.5

# Set the working directory to /app
WORKDIR /app

# Copy the 'application' directory contents into the container at /app
COPY ./application /app

# Make port 8000 available to the world outside this container
EXPOSE 8000

# Execute 'python /app/application.py' when container launches
CMD ["python", "/app/application.py"]
```

## Step 4 - Build the Docker Image

Now that the Dockerfile is ready, you need to build the Docker image. Open a terminal and navigate to the `quickstart_docker` directory. Next, run the following command:

`docker build . -f docker/application/Dockerfile -t exampleapp`

This command will use the instructions in your Dockerfile to build the image. Here is a brief explanation of its arguments:

- `.`: Indicates the build context, which is the current directory.
- `-f docker/application/Dockerfile`: Specifies the path to the Dockerfile.
- `-t exampleapp`: Tags the image with the name `exampleapp` for easier identification later.

## Step 5 - Verify the Docker Image

To verify that the image has been successfully built, list all Docker images on your system by running the following command:

`docker images`

You should see an entry for `exampleapp` in the output, similar to this:

```
  REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
  exampleapp   latest    83ioe0edc28a   2 seconds ago    154MB
  python       3.5       05stv8636w3f   6 weeks ago      154MB
```

## Step 6 - Push the Image to a Repository

To deploy the image to a Kubernetes cluster, you will need to tag it and push it to a container registry such as Docker Hub. You can create a repository on Docker Hub and push the image there by running the following commands:

```
docker tag exampleapp <your-dockerhub-username>/exampleapp
docker push <your-dockerhub-username>/exampleapp
```

Replace `<your-dockerhub-username>` with your actual Docker Hub username.

## Step 7 - Deploy to a Kubernetes cluster

Now that your image is in a repository, you can deploy it to your Kubernetes cluster. Create a file named `deployment.yaml` and add the following lines to it:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exampleapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: exampleapp
  template:
    metadata:
      labels:
        app: exampleapp
    spec:
      containers:
      - name: exampleapp
        image: your-dockerhub-username/exampleapp
        ports:
        - containerPort: 8000
```

Apply this deployment to your Kubernetes cluster:

```
kubectl apply -f deployment.yaml
```

You can learn more about Deployments in Kubernetes on the project's [documentation site](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

## Step 8 - Expose the Application to the World

To expose your application to the world and allow external users to access it from outside the cluster, you can use a Kubernetes Service of [type LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) or [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport). For instance, to define a service of type LoadBalancer for `exampleapp`, create a manifest file named `service.yaml` with the following content:

```
apiVersion: v1
kind: Service
metadata:
  name: exampleapp-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8000
  selector:
    app: exampleapp
```

Read more about how Services work in Kubernetes by clicking [here](https://kubernetes.io/docs/concepts/services-networking/service/#services-in-kubernetes).

## Step 9 - Test the Deployment

To verify if the deployment is working as expected on Kubernetes, you can enter the following commands

- `kubectl get pods`: checks the status of your [pods](https://kubernetes.io/docs/concepts/workloads/pods/).
- `kubectl logs`: displays logs for the `exampleapp` application.

## Conclusion

Congratulations! By now, you should have successfully prepared an application image and deployed it to a Kubernetes cluster. Remember to replace the sample application with your actual production-level application code when you are ready to deploy your own services.
