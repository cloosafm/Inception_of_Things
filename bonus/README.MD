# Bonus : Self-Hosting with GitLab

In this bonus part, we extend Part 3 by self-hosting GitLab within the Kubernetes cluster instead of using GitHub. We install GitLab using Helm charts and configure ArgoCD to sync from the local GitLab instance, demonstrating complete self-hosted CI/CD.

## Deployment files

### 1️⃣ **scripts/setup.sh**

The `setup.sh` script sets up the k3d cluster and ArgoCD (similar to Part 3):
1. **Create k3d cluster**: Sets up a local k3s cluster named "mycluster".
2. **Create namespaces**: Creates `dev`, `argocd`, and `gitlab` namespaces.
3. **Install ArgoCD**: Deploys ArgoCD in the `argocd` namespace.
4. **Wait for ArgoCD**: Ensures ArgoCD pods are ready.
5. **Configure ArgoCD CLI**: Installs the CLI and changes the default admin password.

### 2️⃣ **scripts/gitlab-setup.sh**

The `gitlab-setup.sh` script installs and configures GitLab:
1. **Install Helm**: Ensures Helm is installed for managing GitLab.
2. **Add GitLab Helm repo**: Adds the official GitLab Helm chart repository.
3. **Install GitLab**: Deploys GitLab in minimal mode with LoadBalancer service.
4. **Wait for GitLab**: Ensures GitLab pods are ready.
5. **Get admin password**: Retrieves and displays the GitLab root password.
6. **Port forward**: Exposes GitLab on localhost:8081.

### 3️⃣ **scripts/deploy.sh**

The `deploy.sh` script applies the application manifests:
1. **Apply YAML files**: Deploys the deployment, service, and ArgoCD Application.

### 4️⃣ **confs/deployment.yaml**

Defines a Kubernetes Deployment for the web application (same as Part 3):
- **Namespace**: `dev`
- **Replicas**: 1
- **Image**: `wil42/playground:v1`
- **Port**: 8888

### 5️⃣ **confs/service.yaml**

Creates a Service to expose the application (same as Part 3):
- **Namespace**: `dev`
- **Type**: LoadBalancer
- **Port**: 8888
- **Selector**: Matches the deployment labels

### 6️⃣ **confs/argo-application.yaml**

Configures an ArgoCD Application for continuous delivery from GitLab:
- **Source**: Syncs from a local GitLab repository (URL: `https://gitlab.com/myrepo`)
- **Destination**: Deploys to the `dev` namespace in the k3d cluster
- **Sync Policy**: Automated sync with self-healing and pruning enabled

## ⚙️ Installation

### Prerequisites

Ensure you have the same prerequisites as Part 3 (Docker, k3d, kubectl). Run the prereq.sh from Part 3 if needed.

### Setup the Environment

1. **Run setup script**:
```sh
./scripts/setup.sh
```

This creates the k3d cluster and installs ArgoCD.

2. **Run GitLab setup**:
```sh
./scripts/gitlab-setup.sh
```

This installs GitLab via Helm and sets up port forwarding.

3. **Deploy the application**:
```sh
./scripts/deploy.sh
```

This applies the application manifests.

### Access GitLab

After setup, access GitLab:
- URL: `http://localhost:8081`
- Username: `root`
- Password: Displayed during setup (also retrievable with: `kubectl get secret -n gitlab gitlab-gitlab-initial-root-password -o jsonpath="{.data.password}" | base64 --decode`)

### Setup GitLab Repository

1. Log into GitLab as root.
2. Create a new project (e.g., "myrepo").
3. Add your application code and manifests to the repository.
4. Update the `argo-application.yaml` with the correct GitLab repo URL if different.

### Access ArgoCD

Access the ArgoCD UI:
- Port forward: `kubectl port-forward svc/argocd-server --address 0.0.0.0 -n argocd 8080:443`
- URL: `http://localhost:8080`

Credentials should obviously never be in plain text or within configuration files.
However this project is for demonstration purposes : for easier understanding, credentials can be found here in the steup script.

### Configure ArgoCD for GitLab

In ArgoCD, you may need to:
1. Add the GitLab repository as a source.
2. Update the Application to point to your GitLab repo.
3. Set up authentication if required (e.g., using GitLab tokens).

### Continuous Delivery

With ArgoCD configured for your local GitLab:
1. Push changes to your GitLab repository.
2. ArgoCD will automatically sync and deploy updates.
3. Use GitLab's web interface for version control and CI/CD features.

### Access the Application

Check the application service:
```sh
kubectl get services -n dev
```

Access the app via the LoadBalancer external IP on port 8888.

### Commands

- Check cluster status:
```sh
kubectl get nodes
kubectl get pods -n dev
kubectl get pods -n argocd
kubectl get pods -n gitlab
```

- Access ArgoCD CLI:
```sh
argocd login localhost:8080 --username admin --password admin123 --insecure
```

- List ArgoCD applications:
```sh
argocd app list
```

- Get GitLab password again:
```sh
kubectl get secret -n gitlab gitlab-gitlab-initial-root-password -o jsonpath="{.data.password}" | base64 --decode
```

- Delete the cluster:
```sh
k3d cluster delete mycluster
```