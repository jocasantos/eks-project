# AWS EKS Project

This repository contains the infrastructure and application code for deploying a project on **Amazon Elastic Kubernetes Service (EKS)**. It includes Kubernetes manifests, deployment pipelines, and configuration files.

### Pre-requisites:
- AWS CLI
- kubectl
- eksctl <https://eksctl.io/installation/>

### Installing EKS:
- Install using Fargate
```bash
eksctl create cluster --name my-cluster --region eu-north-1 --fargate
```

- Delete the cluster
```bash
eksctl delete cluster --name my-cluster --region eu-north-1
```

### Configuring kubectl for EKS:
- Update kubeconfig file
```bash
aws eks --region eu-north-1 update-kubeconfig --name my-cluster
```
- Verify the connection
```bash
kubectl get svc
```

### Create a fargate profile:
- Create a fargate profile
```bash
eksctl create fargateprofile \
    --cluster my-cluster \
    --region eu-north-1 \
    --name alb-sample-app \
    --namespace game-2048
```
> Note: This command will create a namespace called `game-2048` besides the fargate profile.

### Deploying the deployment, service and Ingress:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

### Configure IAM OIDC provider:
```bash
export cluster_name=my-cluster
```
- Check if there is an IAM OIDC provider configured already
```bash
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```
- If not:
```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

### Deploying the AWS Load Balancer Controller:
- Download the IAM policy
```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
- Create the IAM policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```
- Create the IAM role
```bash
eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
- Add helm repository
```bash
helm repo add eks https://aws.github.io/eks-charts
```
- Update the repository
```bash
helm repo update eks
```
- Install
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
- Verify that the deployments are running.
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
- Get the Ingress adress
```bash
kubectl get ingress -n game-2048
```
- Now you can check your Load Balancer in your EC2 console.
- Past the Ingress adress in your browser and you should see the 2048 game :D

> Note: Don't forget to delete the resources to avoid charges!




