# Kanban Dashboard – CI/CD Deployment Using Jenkins, Docker & AWS

## Project Overview

This project demonstrates an automated CI/CD pipeline for deploying a React-based Kanban Dashboard application using Docker, Jenkins, and AWS EC2.

The pipeline automatically builds a Docker image, pushes it to Docker Hub, deploys the application to an AWS EC2 instance, and performs health checks. It also includes a basic rollback mechanism to restore the previous working version if a deployment fails.

## Architecture

**Technologies used:**
- AWS EC2 – Cloud hosting
- Ubuntu – Server operating system
- Jenkins – CI/CD automation
- Docker – Containerization
- Docker Hub – Container image registry
- Nginx – Web server
- Git & GitHub – Source code management and webhook integration
- React, TypeScript & Vite – Frontend application

## CI/CD Pipeline

The Jenkins pipeline contains the following stages:

1. **Checkout:** Retrieves the latest source code from GitHub.
2. **Build Docker Image:** Builds the application using a multi-stage Dockerfile.
3. **Push to Docker Hub:** Publishes the versioned image to Docker Hub.
4. **Deploy to EC2:** Pulls the image and deploys the container on AWS EC2.
5. **Health Check:** Verifies container health and HTTP availability.
6. **Rollback:** Restores the previously deployed image if a pipeline failure occurs.

GitHub webhooks automatically trigger the Jenkins pipeline whenever new code is pushed to the main branch.

## Docker Configuration

The application uses a multi-stage Docker build:

- Node.js Alpine image for building the React application.
- Nginx Alpine image for serving the production build.
- Non-root user for running the application.
- Docker HEALTHCHECK for container health monitoring.
- `.dockerignore` to exclude unnecessary files.

## Docker Hub

Docker image repository:

https://hub.docker.com/r/mahaltyhasan/kanban-dashboard

Example image:

`mahaltyhasan/kanban-dashboard:1.0.0`

## AWS Deployment

The application is deployed on an AWS EC2 Ubuntu instance.

**Application URL:** http://13.127.193.22

The application is served through port 80 using an Nginx-based Docker container.

## Rollback Testing

Rollback functionality was tested by temporarily introducing a deliberate pipeline failure after deployment.

The test verified that Jenkins:
- Detected the failed pipeline.
- Removed the newly deployed container.
- Restored the previously deployed Docker image.
- Started the previous container successfully.
- Verified that the restored container was healthy.

The temporary test stage was removed after successful verification.

## Verification Commands

Check running containers:

```bash
docker ps
```

Check container health:

```bash
docker inspect --format='{{.State.Health.Status}}' kanban-app
```

View application logs:

```bash
docker logs kanban-app
```

Test application availability:

```bash
curl -I http://localhost/
```

## Repository Structure

```text
kanban-dashboard/
├── Dockerfile
├── Jenkinsfile
├── nginx.conf
├── .dockerignore
├── package.json
├── package-lock.json
├── src/
├── public/
└── README.md
```

## Learning Outcomes

- Built a multi-stage Docker image.
- Configured Jenkins for automated CI/CD.
- Integrated GitHub webhooks with Jenkins.
- Pushed and pulled Docker images from Docker Hub.
- Deployed a containerized application on AWS EC2.
- Implemented Docker health checks.
- Tested Jenkins failure handling and rollback.

## Author

**Mahalty Hasan**

GitHub: https://github.com/Mahalty-hasan

Project Repository: https://github.com/Mahalty-hasan/kanban-dashboard
