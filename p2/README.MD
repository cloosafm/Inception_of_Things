# P2 : Ingress and Load Balancing

In this part of the project, we build upon the k3s cluster from Part 1 by implementing Ingress and load balancing. We deploy three web applications and configure Ingress rules so that accessing the server's IP address with different HOST headers routes to different applications.

## Deployment files

### 1️⃣ Vagrantfile

The **Vagrantfile** configures a single VM (the server) using Vagrant. It loads configuration from `confs/vars.yaml` and provisions the VM with `scripts/server.sh`. Key configurations include:
- **VM resources**: Memory and CPU allocation for the server.
- **Network**: Private network with a static IP for easy access.
- **Provisioning**: Runs the server script to set up k3s and deploy applications.

### 2️⃣ **confs/vars.yaml**

This YAML file contains variables used by the Vagrantfile and scripts. You can modify settings like VM memory, CPU count, IP address, and script paths before running `vagrant up`. Changes after VM creation require `vagrant reload --provision`.

### 3️⃣ **confs/deployment.yaml**

Defines three Kubernetes Deployments:
- **app1-deployment**: 1 replica of a hello-kubernetes app with message "This is app1".
- **app2-deployment**: 3 replicas of the same app with message "This is app2" (demonstrates load balancing).
- **app3-deployment**: 1 replica with message "This is app3".

### 4️⃣ **confs/service.yaml**

Creates three Services to expose the deployments:
- **app1-service**: Exposes app1-deployment on port 80.
- **app2-service**: Exposes app2-deployment on port 80.
- **app3-service**: Exposes app3-deployment on port 80.

### 5️⃣ **confs/ingress.yaml**

Configures an Ingress resource to route traffic based on HOST headers:
- Requests to `app1.com` route to app1-service.
- Requests to `app2.com` route to app2-service.
- All other requests (default) route to app3-service.

### 6️⃣ **scripts/server.sh**

The `server.sh` script sets up the k3s server and deploys the applications:
1. **Install k3s**: Downloads and installs k3s in server mode with specified network interface and IP.
2. **Generate token**: Copies the node-token for potential future agent nodes.
3. **Update /etc/hosts**: Adds entries for app1.com, app2.com, and app3.com pointing to the server IP.
4. **Deploy Kubernetes resources**: Applies the deployment, service, and ingress YAML files.

## ⚙️ Installation

### Start Vagrant

```sh
vagrant up
```

This creates the server VM with the specified IP address.

### Access the Applications

After provisioning, you can access the apps by setting the HOST header in your requests to the server IP (192.168.56.110 by default). For example:

- `curl -H "Host: app1.com" http://192.168.56.110` → "This is app1"
- `curl -H "Host: app2.com" http://192.168.56.110` → "This is app2" (load balanced across 3 replicas)
- `curl http://192.168.56.110` → "This is app3" (default route)

### Commands

- To SSH into the VM:
```sh
vagrant ssh <VM_NAME>
```

- Check cluster and pods:
```sh
sudo kubectl get nodes
sudo kubectl get pods
sudo kubectl get services
sudo kubectl get ingress
```

- Stop the VM:
```sh
vagrant halt
```

- Restart the VM:
```sh
vagrant up
```

- Destroy the VM:
```sh
vagrant destroy -f
```