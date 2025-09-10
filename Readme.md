# 🚀 Deploying a Two-Tier Application on AWS EKS with Fargate and ALB

![AWS](https://img.shields.io/badge/AWS-Cloud-orange) ![Kubernetes](https://img.shields.io/badge/Kubernetes-K8s-blue) ![Fargate](https://img.shields.io/badge/Fargate-Serverless-purple)

Gamify your DevOps skills! This project demonstrates how to deploy a **two-tier application (2048 game)** on **AWS EKS** using **Fargate** and **AWS Load Balancer Controller**, accessible via DNS while keeping pods secure in private subnets.

---

<details>
<summary>📖 Table of Contents</summary>

1. [Prerequisites](#-prerequisites)  
2. [Step 1: Create EKS Cluster with Fargate](#-step-1-create-eks-cluster-with-fargate)  
3. [Step 2: Create Fargate Profile for App Namespace](#-step-2-create-fargate-profile-for-app-namespace)  
4. [Step 3: Configure kubectl](#-step-3-configure-kubectl)  
5. [Step 4: Deploy Application (Deployment, Service & Ingress)](#-step-4-deploy-application-deployment-service--ingress)  
6. [Step 5: Configure IAM OIDC Provider](#-step-5-configure-iam-oidc-provider)  
7. [Step 6: Setup AWS Load Balancer Controller](#-step-6-setup-aws-load-balancer-controller)  
8. [Step 7: Verify Deployments](#-step-7-verify-deployments)  
9. [Architecture Highlights](#-architecture-highlights)  
10. [Conclusion](#-conclusion)  
11. [References](#-references)

</details>

---

## 📋 Prerequisites

- **AWS CLI** – [Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)  
  ```bash
  aws configure
  ```
kubectl – Installation Guide

eksctl – Installation Guide

Python (Required for AWS CLI)

## Step 1: Create EKS Cluster with Fargate
```bash
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

Private Subnets: Pods run securely

Public Subnets: ALB handles internet traffic

Control Plane: Managed by AWS

## Step 2: Create Fargate Profile for App Namespace
```bash
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```
## Step 3: Configure kubectl
```bash
aws eks update-kubeconfig --name demo-cluster
kubectl get nodes
```
## Step 4: Deploy Application (Deployment, Service & Ingress)
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
Deploys namespace, deployment, service, and ingress
## Step 5: Configure IAM OIDC Provider
```bash
export cluster_name=demo-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
## Step 6: Setup AWS Load Balancer Controller

# Download IAM Policy
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
## Create IAM Policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
## Create IAM Service Account
```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
``Replace with your accound-id``
## Install ALB Controller via Helm
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<your-vpc-id>
```
``Replace with your vpc-id``
## Step 7: Verify Deployments
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller -w
kubectl get ingress -n game-2048
```
Access the app via Ingress DNS
Replace with your screenshot of the app in the browser.

# Architecture Highlights

Private Subnets: Fargate pods (CoreDNS + app) → secure

Public Subnets: ALB → internet-facing

Ingress Resource: Routes traffic to private pods

ALB Annotations:

alb.ingress.kubernetes.io/scheme: internet-facing
alb.ingress.kubernetes.io/target-type: ip

🎯 Conclusion

We successfully deployed a two-tier app on AWS EKS with Fargate, routing traffic securely through a public ALB. Kubernetes concepts (Ingress, private pods, and load balancing) were mapped to real-world cloud architecture, giving practical DevOps skills.

Happyyyyyyyy gamifyingggggggggg!
# References

[AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)

[AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)

[Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
