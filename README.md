# Task Manager Helm Chart

[![Helm](https://img.shields.io/badge/Helm-3.x-blue.svg)](https://helm.sh/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.19+-blue.svg)](https://kubernetes.io/)

A Helm umbrella chart for deploying the Task Manager application on Kubernetes. This chart includes the Task Manager Flask application and MongoDB as dependencies.

## Table of Contents

- [Project Structure](#project-structure)
- [Description](#description)
- [Kubernetes Resources Overview](#kubernetes-resources-overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Upgrading](#upgrading)
- [Accessing the Application](#accessing-the-application)
- [Configuration](#configuration)
- [MongoDB Configuration](#mongodb-configuration)
- [Uninstall](#uninstall)
- [Architecture](#architecture)
- [Troubleshooting](#troubleshooting)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## Project Structure

```
task-manager-umbrella/
├── Chart.yaml                    # Main chart metadata and dependencies
├── Chart.lock                    # Lock file for dependency versions
├── values.yaml                   # Default configuration values
├── charts/
│   ├── app/                      # Subchart for the Task Manager application
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── charts/               # Empty subchart directory
│   │   └── templates/            # Kubernetes templates for the app
│   │       ├── configmap.yaml
│   │       ├── deployment.yaml
│   │       └── service.yaml
│   └── mongodb-15.0.0.tgz        # Downloaded MongoDB Helm chart dependency
├── templates/                    # Main chart templates (empty)
└── README.md                     # This file
```

## Description

The Task Manager is a web application built with Flask that allows users to manage tasks. It uses MongoDB as the database backend and is designed to run in a Kubernetes environment.

## Kubernetes Resources Overview

This Helm chart deploys the following Kubernetes resources:

- **Deployments**: Application pods for the Task Manager Flask app
- **Services**: LoadBalancer service to expose the application externally
- **ConfigMaps**: Configuration data for the application
- **MongoDB StatefulSet**: Database backend with persistent storage (via Bitnami MongoDB subchart)
- **PersistentVolumeClaims**: Storage for MongoDB data persistence (via Bitnami MongoDB subchart)

## Prerequisites

- Kubernetes cluster (version 1.19+)
- Helm 3.x
- kubectl configured to access your cluster

## Cluster Autoscaler Setup (Optional for AWS EKS)

For automatic scaling of EC2 instances in an AWS EKS cluster, install the Cluster Autoscaler using the official Helm chart. This ensures the cluster can scale nodes based on pod resource demands.

1. Add the Cluster Autoscaler repository:
   ```bash
   helm repo add autoscaler https://kubernetes.github.io/autoscaler
   helm repo update
   ```

2. Install or upgrade the Cluster Autoscaler:
   ```bash
   helm upgrade --install cluster-autoscaler autoscaler/cluster-autoscaler \
     --namespace kube-system \
     --set autoDiscovery.clusterName=<your-cluster-name> \
     --set awsRegion=<your-aws-region> \
     --set rbac.serviceAccount.create=true \
     --set rbac.serviceAccount.name=cluster-autoscaler
   ```

   **Parameter explanations**:
   - `autoDiscovery.clusterName`: Your EKS cluster name (automatically discovers all node groups)
   - `awsRegion`: Your AWS region (e.g., `ap-south-1`)
   - `rbac.serviceAccount.*`: Creates a ServiceAccount with appropriate permissions for the autoscaler

This setup eliminates the need for manual YAML configuration and ensures the autoscaler is always up-to-date and secure.

## Installtion

1. Add the Bitnami repository (if not already added):
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```

2. Install the Task Manager chart:
   ```bash
   helm upgrade --install task-manager ./task-manager-umbrella \
     --namespace task-manager \
     --create-namespace \
     --wait
   ```

   This command will:
   - Create the `task-manager` namespace
   - Deploy MongoDB with authentication enabled
   - Deploy the Task Manager application
   - Wait for all resources to be ready

## Upgrading

To upgrade an existing installation:
```bash
helm upgrade task-manager . --namespace task-manager
```

## Accessing the Application

After installation, you can access the Task Manager application using the LoadBalancer service. The application will be available on port 80.

To get the LoadBalancer IP:
```bash
kubectl get svc -n task-manager
```

Look for the `task-manager-service` service and note the `EXTERNAL-IP`. The application will be accessible at `http://<EXTERNAL-IP>`.

## Configuration

The chart can be customized by modifying the `values.yaml` file or by passing values during installation.

### Key Configuration Options

- `replicaCount`: Number of Task Manager application replicas (default: 1)
- `app.image.repository`: Docker image repository for the Task Manager app
- `app.image.tag`: Docker image tag (default: latest)
- `app.mongoUri`: MongoDB connection string
- `app.flaskEnv`: Flask environment (default: production)
- `app.service.type`: Service type (default: LoadBalancer)
- `mongodb.*`: MongoDB configuration options (see Bitnami MongoDB chart documentation)

### MongoDB Configuration

The chart includes MongoDB with the following default settings:
- Architecture: Replicaset with 3 replicas
- Authentication: Enabled with root user `root` and password `secret123`
- Persistence: Enabled with 0.5Gi storage

**Security Note**: Change the default MongoDB credentials in production environments.

## Uninstall

To uninstall the Task Manager:

1. Uninstall the Helm release:
   ```bash
   helm uninstall task-manager --namespace task-manager
   ```

2. Clean up persistent volume claims:
   ```bash
   kubectl delete pvc -n task-manager --all
   ```

3. Delete the namespace (optional):
   ```bash
   kubectl delete namespace task-manager
   ```

## Architecture

This is an umbrella Helm chart that includes:
- **Task Manager App**: A Flask web application containerized and deployed as a Kubernetes deployment
- **MongoDB**: Database backend using the Bitnami MongoDB Helm chart in replicaset mode

The application connects to MongoDB using the configured connection string and stores task data.

## Troubleshooting

- Check pod status: `kubectl get pods -n task-manager`
- View logs: `kubectl logs -n task-manager <pod-name>`
- Check MongoDB connection: Ensure the MongoDB service is running and accessible

## Development

To modify the chart:
1. Update templates in the `charts/app/templates/` directory
2. Modify values in `values.yaml`
3. Test with `helm template .` or install in a development environment

## Contributing

We welcome contributions! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request
