# Amazon EKS Project

This repository contains the infrastructure and application code for deploying a project on **Amazon Elastic Kubernetes Service (EKS)**. It includes Kubernetes manifests, deployment pipelines, and configuration files.

### Pre-requisites
- AWS CLI
- kubectl
- eksctl

### Installing EKS 
- Install using Fargate
```bash
eksctl create cluster --name my-cluster --region eu-north-1 --fargate
```

- Delete the cluster
```bash
eksctl delete cluster --name my-cluster --region eu-north-1
```



