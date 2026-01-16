# KubeSSH

Helm chart for deploying OpenSSH server on Kubernetes.

## Overview

This repository contains a Helm chart that deploys an OpenSSH server on Kubernetes, allowing secure SSH access to a containerized environment. Perfect for development, testing, or providing SSH access to Kubernetes workloads.

## Features

- 🚀 Easy deployment of OpenSSH server on Kubernetes
- 🔑 Support for both password and SSH key authentication
- 💾 Optional persistent storage for SSH host keys and user data
- ⚙️ Highly configurable via Helm values
- 🔒 Security-focused with sensible defaults
- 📦 Uses the reliable [linuxserver/openssh-server](https://github.com/linuxserver/docker-openssh-server) Docker image

## Quick Start

### Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

### Installation

1. Clone this repository:
```bash
git clone https://github.com/leg100/kubessh.git
cd kubessh
```

2. Install the chart:
```bash
helm install my-ssh ./kubessh
```

3. Follow the post-installation notes to connect via SSH.

### Basic Example

Deploy an SSH server with a user named "developer" and password authentication:

```bash
helm install my-ssh ./kubessh \
  --set ssh.user.name=developer \
  --set ssh.user.password=SecurePassword123
```

### Recommended Example (SSH Key Authentication)

For production use, SSH key authentication is recommended:

```bash
helm install my-ssh ./kubessh \
  --set ssh.user.name=developer \
  --set ssh.user.publicKeys[0]="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ..." \
  --set ssh.config.passwordAuthentication=false \
  --set ssh.persistence.enabled=true
```

## Documentation

For detailed configuration options and examples, see the [Helm chart README](kubessh/README.md).

## Configuration Highlights

### Service Types

- **LoadBalancer** (default): Exposes SSH via cloud provider load balancer
- **NodePort**: Exposes SSH on a static port on each node
- **ClusterIP**: Internal access only (use port-forward for external access)

### Authentication

- **SSH Keys** (recommended): Secure, passwordless authentication
- **Password**: Simple but less secure
- **Both**: Support both methods simultaneously

### Persistence

Enable persistent storage to maintain SSH host keys across pod restarts:

```bash
helm install my-ssh ./kubessh --set ssh.persistence.enabled=true
```

This prevents "host key changed" warnings when pods restart.

## Use Cases

- **Development environments**: Provide SSH access to containerized development environments
- **Jump hosts**: Use as a secure jump host within your Kubernetes cluster
- **CI/CD pipelines**: Enable SSH access for deployment and debugging
- **Testing**: Quickly spin up temporary SSH servers for testing

## Connecting to Your SSH Server

After installation, get connection details:

```bash
# For LoadBalancer
export SERVICE_IP=$(kubectl get svc my-ssh-kubessh -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
ssh developer@$SERVICE_IP

# For NodePort
export NODE_PORT=$(kubectl get svc my-ssh-kubessh -o jsonpath='{.spec.ports[0].nodePort}')
export NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
ssh developer@$NODE_IP -p $NODE_PORT

# For ClusterIP (using port-forward)
kubectl port-forward svc/my-ssh-kubessh 22:22
ssh developer@localhost -p 22
```

## Uninstalling

```bash
helm uninstall my-ssh
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the Apache License 2.0.

## Credits

This chart uses the excellent [linuxserver/openssh-server](https://github.com/linuxserver/docker-openssh-server) Docker image maintained by the LinuxServer.io team.

