# Toktik

## Project Description

**Toktik** is an ambitious project with the goal of developing a highly scalable and efficient video service platform. Employing microservice architecture and deploying on a Kubernetes cluster, this project encompasses a range of learning objectives, including:

1. **Building a Microservice-Style System**: Our approach involves decomposing the system into appropriately designed microservices, allowing for seamless scalability and streamlined maintenance.

2. **Synchronous and Asynchronous Communication**: We implement both synchronous and asynchronous communication patterns, ensuring effective handling of a variety of tasks and processes.

## System Requirements

At the heart of **Toktik** is a video service platform designed to empower users to log in, view, manage, and upload their videos. Key requirements include:

- **Video Management**: Users can effortlessly upload and manage their videos, with a restriction that videos exceeding one minute in length will be automatically rejected.

- **Video Offloading**: To ensure efficiency, video uploads are offloaded to an S3-compatible service. This approach prevents user videos from being unnecessarily uploaded through the backend gateway.

- **Video Processing**: Our system processes videos and efficiently chunks them for streaming using the HLS format. This enables smooth video streaming and playback.

- **Thumbnail Generation**: For an enhanced user experience, we extract thumbnail images from videos, providing a quick preview of the content.

- **Microservices and Kubernetes Deployment**: **Toktik** embraces a microservices paradigm and deploys on a Kubernetes cluster. This approach guarantees scalability, reliability, and efficient resource management.

- **S3 Storage**: File objects are securely stored using S3, offering a scalable and resilient storage solution.

## Non-Functional Requirements

In our commitment to excellence, we address various non-functional requirements, ensuring the system's stability, security, and efficiency:

- **Secure S3 URLs**: We enhance security by utilizing presigned URLs to secure S3 resources, safeguarding sensitive content and data.

- **Scalability**: Each microservice/component is designed to scale independently, allowing the system to handle high loads without performance bottlenecks.

- **Authentication**: We implement robust authentication methods that don't compromise the performance of our services. User access control and data security are paramount.

- **GitHub Workflows**: We streamline the deployment process by creating GitHub workflows for building container images and efficiently uploading them to a container registry service.

**Toktik** merges the principles of microservices, Kubernetes, and cutting-edge cloud technologies. Our goal is to provide a robust video service platform that caters to both users and administrators. By creating a scalable and secure environment for video management and streaming, we are committed to delivering an exceptional experience for all stakeholders.

&nbsp;

# Feature Checklist

## System Requirements

- [x] **Video Management**: Users can effortlessly upload and manage their videos, with a restriction that videos exceeding one minute in length will be automatically rejected.

- [x] **Video Offloading**: To ensure efficiency, video uploads are offloaded to an S3-compatible service. This approach prevents user videos from being unnecessarily uploaded through the backend gateway.

- [x] **Video Processing**: Our system processes videos and efficiently chunks them for streaming using the HLS format. This enables smooth video streaming and playback.

- [x] **Thumbnail Generation**: For an enhanced user experience, we extract thumbnail images from videos, providing a quick preview of the content.

- [x] **Microservices and Kubernetes Deployment**: **Toktik** embraces a microservices paradigm and deploys on a Kubernetes cluster. This approach guarantees scalability, reliability, and efficient resource management.

- [x] **S3 Storage**: File objects are securely stored using S3, offering a scalable and resilient storage solution.

## Non-Functional Requirements

- [x] **Secure S3 URLs**: We enhance security by utilizing presigned URLs to secure S3 resources, safeguarding sensitive content and data.

- [x] **Scalability**: Each microservice/component is designed to scale independently, allowing the system to handle high loads without performance bottlenecks.

- [x] **Authentication**: We implement robust authentication methods that don't compromise the performance of our services. User access control and data security are paramount.

- [x] **GitHub Workflows**: We streamline the deployment process by creating GitHub workflows for building container images and efficiently uploading them to a container registry service.

&nbsp;

# Toktik Deployment Guide

This guide outlines the deployment process for the Toktik project on a Kubernetes cluster. We will start by deploying RabbitMQ and then proceed with other components.

## Prerequisites

Before you begin, ensure that you have the following prerequisites in place:

- A functioning Kubernetes cluster.
- `kubectl` command-line tool properly configured to work with your cluster.
- You have [traefik](https://doc.traefik.io/traefik/getting-started/install-traefik/) installed and running

&nbsp;

## Cloning the Deployment Repository

Begin by cloning the Toktik deployment repository and organizing the file tree as follows:

```bash
# Clone the deployment repository
git clone https://github.com/polpon/toktik-deployment.git

# Move into the cloned repository
cd toktik-deployment

# Your file tree should look like this:
#
# toktik-deployment
# ├── README.md
# └── k8s
#     ├── db-deployment.yaml
#     ├── db-service.yaml
#     ├── dockerconfigjson.yaml
#     ├── pvc.yaml
#     ├── rabbitmq
#     │   ├── cluster-operator.yml
#     │   └── rabbitmq.yaml
#     ├── toktik-backend-deployment.yaml
#     ├── toktik-backend-service.yaml
#     ├── toktik-chunker-deployment.yaml
#     ├── toktik-chunker-service.yaml
#     ├── toktik-convert-deployment.yaml
#     ├── toktik-convert-service.yaml
#     ├── toktik-dead-letter-deployment.yaml
#     ├── toktik-dead-letter-service.yaml
#     ├── toktik-frontend-deployment.yaml
#     ├── toktik-frontend-service.yaml
#     ├── toktik-ingress.yaml
#     ├── toktik-manager-deployment.yaml
#     ├── toktik-manager-service.yaml
#     ├── toktik-thumbnail-deployment.yaml
#     └── toktik-thumbnail-service.yaml
```

&nbsp;

## RabbitMQ Deployment

Firstly, let's deploy RabbitMQ:

```bash
# Enter 'k8s' folder
cd k8s

# Start rabbitmq cluster
kubectl apply -f rabbitmq/cluster-operator.yml

# Then create the cluster object named 'rabbit-mq'
kubectl apply -f rabbitmq/rabbitmq.yaml
```

&nbsp;

## Secret Configuration
Before you could pull the image from the github package, you first need a correct secret configuration for k8s.

https://dev.to/asizikov/using-github-container-registry-with-kubernetes-38fb

#### TDLR:
create & apply the secret out right
```bash
kubectl create secret docker-registry dockerconfigjson-github-com --docker-server=ghcr.io --docker-username={Your username} --docker-password={Your docker PAT (personal access token)}
```
OR create the config file so we could apply it later
```bash
kubectl create secret docker-registry dockerconfigjson-github-com --docker-server=ghcr.io --docker-username={Your username} --docker-password={Your docker PAT (personal access token)} --dry-run=client --output=yaml > ghsecret.yaml
```

&nbsp;

## Apply the rest of the services and deployment
Make sure [traefik](https://doc.traefik.io/traefik/getting-started/install-traefik/) is installed and running

```bash
# Inside 'k8s' folder
kubectl apply -f .
```

Check if all the processes is pulled successfully
```bash
watch -n 1 kubectl get all
```

Should take ~30secs - 1min for all services to start, then voila. Toktik is now live at http://localhost

&nbsp;

## Deleting Deployments
Deleting toktik services
```bash
# Inside 'k8s' folder
kubectl delete -f .
```

Deleting the Rabbitmq Cluster Object and Cluster itself
```bash
# Delete cluster object
kubectl delete rabbitmqclusters.rabbitmq.com rabbit-mq

# Delete cluster
kubectl delete -f rabbitmq/cluster-operator.yml
```
