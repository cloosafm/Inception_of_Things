# P3 : Continuous Delivery with ArgoCD

In this part of the project, we shift from Vagrant to k3d (k3s in Docker) for a more lightweight setup. We use ArgoCD to implement continuous delivery, automatically deploying updates to a web application from a GitHub repository. This allows for easy version management and rollbacks.

## Deployment files

### 1️⃣ **scripts/prereq.sh**

The `prereq.sh` script ensures all necessary tools are installed on your local machine:
1. **Install system packages**: Updates the system and installs required packages like curl, wget, git, etc.
2. **Install Docker**: Sets up Docker and adds the user to the docker group.
3. **Install k3d**: Downloads and installs k3d for running k3s clusters in Docker.
4. **Install kubectl**: Downloads and installs the Kubernetes CLI tool.
5. **Configure aliases**: Adds kubectl alias and bash completion.

### 2️⃣ **scripts/setup.sh**

The `setup.sh` script creates the k3d cluster and configures ArgoCD:
1. **Create k3d cluster**: Sets up a local k3s cluster named "mycluster".
2. **Install ArgoCD**: Deploys ArgoCD in the `argocd` namespace.
3. **Wait for ArgoCD**: Ensures ArgoCD pods are ready.
4. **Configure ArgoCD CLI**: Installs the ArgoCD CLI and changes the default admin password.
5. **Create dev namespace**: Sets up a `dev` namespace for the application.
6. **Apply YAML files**: Deploys the application manifests and ArgoCD Application.

### 3️⃣ **conf/deployment.yaml**

Defines a Kubernetes Deployment for the web application:
- **Namespace**: `dev`
- **Replicas**: 1
- **Image**: `wil42/playground:v1`
- **Port**: 8888

### 4️⃣ **conf/service.yaml**

Creates a Service to expose the application:
- **Namespace**: `dev`
- **Type**: LoadBalancer
- **Port**: 8888
- **Selector**: Matches the deployment labels

### 5️⃣ **conf/argo-application.yaml**

Configures an ArgoCD Application for continuous delivery:
- **Source**: Syncs from a GitHub repository (configured for the project's repo)
- **Destination**: Deploys to the `dev` namespace in the k3d cluster
- **Sync Policy**: Automated sync with self-healing and pruning enabled

## ⚙️ Installation

### Prerequisites

Run the prerequisites script to set up your environment:

```sh
./scripts/prereq.sh
```

This installs Docker, k3d, kubectl, and other required tools.

### Setup the Environment

Run the setup script to create the cluster and deploy ArgoCD:

```sh
./scripts/setup.sh
```

This will:
- Create a k3d cluster
- Install and configure ArgoCD
- Deploy the application

### Access ArgoCD

After setup, you can access the ArgoCD UI:
- Port forward: `kubectl port-forward svc/argocd-server --address 0.0.0.0 -n argocd 8080:443`
- URL: `http://localhost:8080`

Credentials should obviously never be in plain text or within configuration files.
However this project is for demonstration purposes : for easier understanding, credentials can be found here in the steup script.

### Access the Application

The application should be accessible via the LoadBalancer service. Check the service:

```sh
kubectl get services -n dev
```

Look for the external IP and access the app on port 8888.

### Continuous Delivery

With ArgoCD configured, any changes pushed to the GitHub repository will automatically trigger a deployment. To demonstrate CD:
1. Update the image version in the repository's deployment.yaml
2. Push the changes
3. ArgoCD will automatically sync and update the deployment

### Rollback

To rollback to a previous version:
1. In the ArgoCD UI, select the application
2. Go to the History tab
3. Select a previous deployment and click "Rollback"

### Commands

- Check cluster status:
```sh
kubectl get nodes
kubectl get pods -n dev
kubectl get pods -n argocd
```

- Access ArgoCD CLI:
```sh
argocd login localhost:8080 --username admin --password admin123 --insecure
```

- List ArgoCD applications:
```sh
argocd app list
```

- Delete the cluster:
```sh
k3d cluster delete mycluster
```