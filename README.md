# AWS-EKS-Deployment-Pipeline

## Pipeline 1: Creating the Cluster

- Authenticate into AWS
- Create the Kubernetes Cluster
- Creating a configuration file for kubectl cluster

## Pipeline 2: Deploying the Containers

- Linting
- Build the docker image and uploading image to docker hub repository
- Set current kubectl context to cluster
- Create the Blue and Green replication controller with their docker image
- Create route traffic service and exposing Blue replication
- Wait until user instruction
- Update the service to Green

## Dependencies

- Jenkins
- Blue Ocean and Pipeline-AWS Plugin in Jenkins
- Docker and Kubectl
- AWS Cli
