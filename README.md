# Kubernetes EKS Cluster with Datadog

This repository contains configuration for setting up an Amazon EKS (Elastic Kubernetes Service) cluster with Datadog monitoring integration and deploying Rust-based microservices.

## Overview

The project provides a simple configuration for creating and managing an EKS cluster on AWS. The cluster is configured with ARM-based instances for better performance and cost efficiency. It includes Datadog for monitoring and two Rust microservices that demonstrate distributed tracing.

## Prerequisites

- [AWS CLI](https://aws.amazon.com/cli/) installed and configured
- [eksctl](https://eksctl.io/) installed
- AWS account with appropriate permissions
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed (for cluster management)
- [Helm](https://helm.sh/docs/intro/install/) installed (for Datadog installation)
- Datadog account with API key

## Cluster Configuration

The cluster is configured with the following specifications:
- Cluster name: `sandbox`
- Region: `us-west-2`
- Node group: ARM-based instances (`m6g.large`)
- Desired capacity: 2 nodes

## Complete Installation Guide

### 1. Creating the Cluster

To create the EKS cluster using the provided configuration:

```bash
eksctl create cluster -f cluster/cluster-config.yaml
```

This process may take 15-20 minutes to complete.

### 2. Connecting to the Cluster

After the cluster is created, configure kubectl to connect to it:

```bash
aws eks update-kubeconfig --name sandbox --region us-west-2
```

### 3. Verifying the Cluster

Check if the nodes are running:

```bash
kubectl get nodes
```

You should see two nodes with status "Ready".

### 4. Creating the Namespace

Create the namespace for the Rust services:

```bash
kubectl apply -f namespaces/rust-services-namespace.yaml
```

Verify the namespace was created:

```bash
kubectl get namespaces
```

### 5. Installing Datadog

There are two options for installing Datadog:

#### Using Helm 

1. Create a Kubernetes secret with your Datadog API key:

```bash
kubectl create secret generic datadog-secret --from-literal api-key=<YOUR_DATADOG_API_KEY>
```

2. Add the Datadog Helm repository and update:

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

3. Install Datadog using Helm:

```bash
helm install datadog-agent -f datadog/datadog-values.yaml datadog/datadog
```

4. Verify Datadog is running:

```bash
kubectl get pods -l app.kubernetes.io/name=datadog
```

### 6. Deploying the Services

Deploy the services in the correct order (service-a first, then service-b):

```bash
kubectl apply -f services/service-a.yaml
kubectl apply -f services/service-b.yaml
```

Verify the services are running:

```bash
kubectl get pods -n rust-services
kubectl get services -n rust-services
```

### 7. Accessing the Services

To access the services from your local machine, you can use port forwarding:

```bash
# For service-a
kubectl port-forward -n rust-services svc/service-a 3000:80

# For service-b (in a separate terminal)
kubectl port-forward -n rust-services svc/service-b 3001:80
```

Then you can access:
- Service A: http://localhost:3000
- Service B: http://localhost:3001

## Monitoring in Datadog

After deploying the services, you can view:
- Traces in the APM section of your Datadog dashboard
- Logs in the Logs section
- Infrastructure metrics in the Infrastructure section

## Deleting the Cluster

When you're done with the cluster, you can delete it to avoid incurring unnecessary costs:

```bash
eksctl delete cluster -f cluster/cluster-config.yaml
```

## License

This project is licensed under the FreeBSD License - see the [LICENSE](LICENSE) file for details.

## Contributing

We welcome community contributions and pull requests. 
