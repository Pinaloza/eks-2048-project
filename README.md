# Amazon EKS 2048 Game Deployment

## Project Overview

This project demonstrates deploying a containerized 2048 game application on Amazon Elastic Kubernetes Service (EKS) using Kubernetes.

The project includes:

* Amazon EKS Cluster
* AWS Load Balancer Controller
* Kubernetes Deployment
* Kubernetes Service
* Kubernetes Ingress
* Application Load Balancer (ALB)

The application is exposed to the internet through an AWS Application Load Balancer managed by Kubernetes Ingress resources.

---

## Architecture

User → AWS ALB → Kubernetes Ingress → Service → 2048 Game Pods

---

## Prerequisites

Before starting, install the following tools:

* AWS CLI
* kubectl
* eksctl
* Helm

Verify installation:

```bash
aws --version
kubectl version --client
eksctl version
helm version
```

Configure AWS credentials:

```bash
aws configure
```

---

## Step 1: Create EKS Cluster

```bash
eksctl create cluster \
  --name demo-cluster \
  --region us-east-1 \
  --fargate
```

Verify cluster creation:

```bash
kubectl get nodes
```

---

## Step 2: Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster demo-cluster \
  --approve
```

---

## Step 3: Download IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

Create IAM Policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

## Step 4: Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=<POLICY_ARN> \
  --approve
```

---

## Step 5: Install AWS Load Balancer Controller

Add Helm repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Install controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

Verify:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

---

## Step 6: Deploy Application

Apply deployment manifest:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get pods
```

---

## Step 7: Create Service

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc
```

---

## Step 8: Create Ingress

```bash
kubectl apply -f ingress.yaml
```

Verify:

```bash
kubectl get ingress
```

Wait a few minutes for the AWS Application Load Balancer to be created.

---

## Validation

Verify resources:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

Access the application using the ALB DNS name.

---

## Technologies Used

* AWS EKS
* Kubernetes
* AWS IAM
* AWS Application Load Balancer
* AWS Load Balancer Controller
* eksctl
* kubectl
* Helm

---

## Learning Outcomes

* Created and managed an Amazon EKS cluster
* Configured IAM Roles for Service Accounts (IRSA)
* Installed AWS Load Balancer Controller
* Deployed applications on Kubernetes
* Exposed applications using Ingress and ALB
* Managed Kubernetes resources using kubectl

```
```
