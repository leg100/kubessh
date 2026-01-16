# KubeSSH Helm Chart

A Helm chart for deploying an OpenSSH server on Kubernetes.

## Introduction

This chart deploys an OpenSSH server on a Kubernetes cluster using the [linuxserver/openssh-server](https://github.com/linuxserver/docker-openssh-server) Docker image.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installing the Chart

To install the chart with the release name `my-ssh`:

```bash
helm install my-ssh ./kubessh
```

The command deploys OpenSSH on the Kubernetes cluster with default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-ssh` deployment:

```bash
helm uninstall my-ssh
```

## Configuration

The following table lists the configurable parameters of the KubeSSH chart and their default values.

### Global Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | OpenSSH server image repository | `linuxserver/openssh-server` |
| `image.tag` | Image tag (overrides Chart.appVersion) | `""` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full chart name | `""` |

### Service Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `LoadBalancer` |
| `service.port` | Service port | `22` |
| `service.nodePort` | NodePort (if service type is NodePort or LoadBalancer) | `""` |

### SSH Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ssh.user.name` | SSH user name | `user` |
| `ssh.user.password` | SSH user password (not recommended, use publicKeys) | `""` |
| `ssh.user.publicKeys` | List of SSH public keys for authentication | `[]` |
| `ssh.sudoAccess` | Enable sudo access for the user | `false` |
| `ssh.config.passwordAuthentication` | Allow password authentication | `true` |
| `ssh.config.pubkeyAuthentication` | Allow public key authentication | `true` |
| `ssh.config.permitRootLogin` | Permit root login | `false` |
| `ssh.config.gatewayPorts` | Enable gateway ports | `false` |
| `ssh.env` | Additional environment variables | `[]` |

### Persistence Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ssh.persistence.enabled` | Enable persistent volume for SSH data | `false` |
| `ssh.persistence.existingClaim` | Use existing PVC | `""` |
| `ssh.persistence.storageClass` | Storage class | `""` |
| `ssh.persistence.accessMode` | Access mode | `ReadWriteOnce` |
| `ssh.persistence.size` | Volume size | `1Gi` |
| `ssh.persistence.mountPath` | Mount path for persistent data | `/config` |

### Other Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |
| `podAnnotations` | Pod annotations | `{}` |
| `podLabels` | Pod labels | `{}` |
| `podSecurityContext` | Pod security context | `{}` |
| `securityContext` | Container security context | `{}` |
| `resources` | Resource requests/limits | `{}` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |

## Examples

### Basic Installation with Password Authentication

```bash
helm install my-ssh ./kubessh \
  --set ssh.user.name=myuser \
  --set ssh.user.password=mypassword
```

### Installation with SSH Key Authentication (Recommended)

```bash
helm install my-ssh ./kubessh \
  --set ssh.user.name=myuser \
  --set ssh.user.publicKeys[0]="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ..." \
  --set ssh.config.passwordAuthentication=false
```

### Installation with Persistence

```bash
helm install my-ssh ./kubessh \
  --set ssh.user.name=myuser \
  --set ssh.user.publicKeys[0]="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ..." \
  --set ssh.persistence.enabled=true \
  --set ssh.persistence.size=5Gi
```

### Installation with NodePort Service

```bash
helm install my-ssh ./kubessh \
  --set service.type=NodePort \
  --set service.nodePort=30022 \
  --set ssh.user.name=myuser \
  --set ssh.user.password=mypassword
```

### Installation with Custom Resources

```bash
helm install my-ssh ./kubessh \
  --set ssh.user.name=myuser \
  --set ssh.user.password=mypassword \
  --set resources.requests.memory=256Mi \
  --set resources.requests.cpu=100m \
  --set resources.limits.memory=512Mi \
  --set resources.limits.cpu=200m
```

## Connecting to SSH

After installation, follow the instructions shown in the NOTES output to get the SSH connection details.

For LoadBalancer service:
```bash
export SERVICE_IP=$(kubectl get svc --namespace default my-ssh-kubessh --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
ssh myuser@$SERVICE_IP -p 22
```

For NodePort service:
```bash
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services my-ssh-kubessh)
export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
ssh myuser@$NODE_IP -p $NODE_PORT
```

For ClusterIP service (using port-forward):
```bash
kubectl port-forward svc/my-ssh-kubessh 22:22
ssh myuser@localhost -p 22
```

## Security Considerations

1. **Use SSH Keys**: Always prefer SSH key authentication over password authentication in production.
2. **Enable Persistence**: Enable persistence to maintain SSH host keys across pod restarts, preventing "host key changed" warnings.
3. **Secure Storage**: Store passwords and sensitive configuration in Kubernetes Secrets rather than in values files.
4. **Network Policies**: Consider implementing network policies to restrict SSH access.
5. **Resource Limits**: Set appropriate resource limits to prevent resource exhaustion.

## Troubleshooting

### Pod fails to start
Check the pod logs:
```bash
kubectl logs -l app.kubernetes.io/name=kubessh
```

### Cannot connect via SSH
1. Verify the service is running:
   ```bash
   kubectl get svc
   ```
2. Check if the LoadBalancer has an external IP (if using LoadBalancer):
   ```bash
   kubectl get svc my-ssh-kubessh
   ```
3. Verify pod is ready:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=kubessh
   ```

### Host key verification failed
This typically happens when:
- The pod restarts without persistence enabled
- The PVC was deleted and recreated

Enable persistence to maintain SSH host keys:
```bash
helm upgrade my-ssh ./kubessh --set ssh.persistence.enabled=true
```

## License

This chart is licensed under the Apache License 2.0.
