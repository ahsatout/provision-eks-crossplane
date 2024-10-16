# Crossplane: Create AWS VPC - EKS - IRSA - Cluster Autoscaler - CSI Driver

## Install Crossplane on Kubernetes

```bash
minikube start --driver=docker
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm search repo crossplane
helm install crossplane \
    --namespace crossplane-system \
    --create-namespace \
    --version 1.13.2 \
    crossplane-stable/crossplane

kubectl get pods -n crossplane-system
kubectl get crds | grep crossplane.io
```

## Create AWS VPC

- VPC resource: Defines the main VPC network.
- Internet Gateway: Allows communication between the VPC and the internet.
- Subnets: Subdivisions of the VPC network.
- Route Tables: Define how network traffic is directed.
- Route Table Associations: Link subnets to route tables.

```bash
kubectl apply -f 0-crossplane/2-provider-aws-ec2.yaml
kubectl get providers
kubectl apply -f 2-vpc
kubectl get VPC
kubectl get InternetGateway
kubectl get RouteTableAssociation
```

## Create EKS Cluster

- EKS Cluster: The main Kubernetes control plane managed by AWS.
- Node Group: A group of EC2 instances that serve as Kubernetes worker nodes.

```bash
kubectl apply -f 0-crossplane
kubectl get providers
kubectl apply -f 3-eks
kubectl get Cluster
kubectl get NodeGroup
aws configure --profile crossplane
aws eks update-kubeconfig --name dev-cluster --region us-east-2 --profile crossplane
kubectl get nodes
```

## Create OpenID Connect Provider (OIDC)

OpenID Connect (OIDC) Provider: Links AWS IAM with Kubernetes service accounts.
IAM Roles: Define permissions for Kubernetes pods.
Service Accounts: Kubernetes objects that provide an identity for processes running in a pod.

```bash
kubectl apply -f 4-irsa
kubectl get OpenIDConnectProvider
kubectl get Addon
```

## Deploy EBS CSI driver

EBS CSI Driver Addon: Installs the necessary components to use EBS volumes in Kubernetes.
Storage Class: Defines the parameters for dynamically provisioning EBS volumes.

```bash
kubectl apply -f 5-storageclass
```

## Deploy Cluster Autoscaler

- Cluster Autoscaler Deployment: Runs the autoscaler logic within the cluster.
- IAM Role: Provides necessary permissions for the autoscaler to interact with AWS.
- Service Account: Links the Kubernetes deployment with the IAM role (using IRSA).

```bash
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm search repo cluster-autoscaler

helm install autoscaler \
    --namespace kube-system \
    --version 9.29.3 \
    --values 6-cluster-autoscaler/1-helm-values.yaml \
    autoscaler/cluster-autoscaler
```
