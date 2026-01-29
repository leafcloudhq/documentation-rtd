# Private LLM with Open WebUI and Ollama

In this tutorial, you'll deploy a fully functional private LLM environment using Open WebUI and Ollama on your Leafcloud Kubernetes cluster. Open WebUI provides a ChatGPT-like interface, while Ollama handles running the AI models on your GPU.

## Prerequisites

- A [Kubernetes cluster on Gardener](../gardener/creating-a-kubernetes-cluster.md) with GPU-enabled worker nodes
- [Helm](../kubernetes/installing-helm.md) installed on your local machine
- [kubectl](../gardener/gardenctl.md) configured to access your cluster
- A domain name (or use Gardener's built-in DNS)

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                    │
│                                                         │
│  ┌─────────────┐      ┌─────────────┐                  │
│  │  Open WebUI │─────▶│   Ollama    │                  │
│  │   (Web UI)  │      │ (LLM Engine)│                  │
│  └──────┬──────┘      └──────┬──────┘                  │
│         │                    │                          │
│         │              ┌─────┴─────┐                   │
│         │              │    GPU    │                   │
│         │              └───────────┘                   │
│  ┌──────┴──────┐                                       │
│  │   Ingress   │◀── TLS (Let's Encrypt)               │
│  └──────┬──────┘                                       │
└─────────┼───────────────────────────────────────────────┘
          │
          ▼
    Your Browser
```

## Step 1: Install the NVIDIA GPU Operator

Before deploying Ollama, you need the NVIDIA GPU Operator to make GPUs available to your pods.

Follow the guide: [Installing the NVIDIA GPU Operator](../kubernetes/installing-nvidia-gpu-operator.md)

Once completed, verify the GPU is detected:

```bash
kubectl describe nodes | grep -A5 "nvidia.com/gpu"
```

## Step 2: Install Ingress and TLS

Before deploying Open WebUI, you need NGINX Ingress Controller and cert-manager for TLS certificates.

Follow the guide: [Ingress with NGINX and Let's Encrypt](../kubernetes/ingress-with-nginx-letsencrypt.md)

This will set up:

- NGINX Ingress Controller
- cert-manager
- ClusterIssuer for Let's Encrypt

Once completed, verify everything is ready:

```bash
kubectl get clusterissuer letsencrypt-prod
kubectl get pods -n ingress-nginx
kubectl get pods -n cert-manager
```

## Step 3: Deploy Open WebUI with Ollama

Add the Open WebUI Helm repository:

```bash
helm repo add open-webui https://helm.openwebui.com/
helm repo update
```

Create a file named `values.yaml` with the following configuration:

```yaml
replicaCount: 1

service:
  type: ClusterIP
  port: 8080

image:
  repository: ghcr.io/open-webui/open-webui
  tag: latest

# Ollama Configuration
ollama:
  enabled: true
  fullnameOverride: "open-webui-ollama"

  ollama:
    gpu:
      enabled: true
      type: 'nvidia'
      number: 1

  resources:
    requests:
      nvidia.com/gpu: 1
      memory: 12Gi
      cpu: 4
    limits:
      nvidia.com/gpu: 1
      memory: 12Gi
      cpu: 4

  nodeSelector:
    nvidia.com/gpu.present: "true"

  tolerations:
    - key: "nvidia.com/gpu"
      operator: "Exists"
      effect: "NoSchedule"

  service:
    type: ClusterIP
    port: 11434

  persistentVolume:
    enabled: true
    size: 200Gi
    accessModes:
      - ReadWriteOnce

# Open WebUI Configuration
openwebui:
  env:
    - name: OLLAMA_BASE_URL
      value: http://open-webui-ollama:11434
    - name: ENABLE_SIGNUP
      value: "true"
    - name: ENABLE_API_KEYS
      value: "true"
    - name: WEBUI_SECRET_KEY
      value: "change-this-to-a-secure-random-string"

persistence:
  enabled: true
  size: 5Gi
  accessModes:
    - ReadWriteOnce

# Ingress Configuration
ingress:
  enabled: true
  class: "nginx"
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: letsencrypt-prod
    acme.cert-manager.io/http01-ingress-class: nginx
    kubernetes.io/tls-acme: "true"
    # Gardener DNS annotations - see docs for details
    dns.gardener.cloud/dnsnames: openwebui.your-project.xxxx.gardener.leaf.cloud
    dns.gardener.cloud/ttl: "600"
    dns.gardener.cloud/class: garden

  host: openwebui.your-project.xxxx.gardener.leaf.cloud  # Replace with your domain
  tls: true
```

For details on configuring Gardener DNS, see [Using Gardener DNS with Ingress](../gardener/gardener-dns.md).

!!! warning "Important"
    Replace the following values before deploying:

    - `WEBUI_SECRET_KEY`: Use a secure random string
    - `host` and `dns.gardener.cloud/dnsnames`: Your Gardener shoot domain (find it in the Gardener dashboard under Infrastructure > Shoot Domain)

Deploy the Helm chart:

```bash
helm install open-webui open-webui/open-webui \
  --namespace ai \
  --create-namespace \
  --values values.yaml
```

## Step 4: Verify the Deployment

Check that all pods are running:

```bash
kubectl get pods -n ai
```

You should see:

```
NAME                          READY   STATUS    RESTARTS   AGE
open-webui-0                  1/1     Running   0          2m
open-webui-ollama-0           1/1     Running   0          2m
```

Check the certificate is issued:

```bash
kubectl get certificate -n ai
```

## Step 5: Access Open WebUI

Open your browser and navigate to your domain (e.g., `https://openwebui.<cluster-name>.<project-id>.gardener.leaf.cloud`).

1. **Create an account**: The first user to sign up becomes the admin
2. **Download a model**: Click on your profile > Settings > Models > Pull a model
3. **Start chatting**: Select a model and begin your conversation

### Recommended Models to Start

| Model | Size | Best For |
|-------|------|----------|
| `llama3.2:3b` | 2GB | Quick responses, testing |
| `llama3.1:8b` | 4.7GB | General purpose, good balance |
| `codellama:13b` | 7GB | Code generation |
| `mistral:7b` | 4GB | Fast, high quality |

To pull a model via CLI:

```bash
kubectl exec -it -n ai open-webui-ollama-0 -- ollama pull llama3.1:8b
```

## Upgrading

To upgrade your deployment with new values:

```bash
helm upgrade open-webui open-webui/open-webui \
  --namespace ai \
  --values values.yaml
```

## Troubleshooting

### Pods stuck in Pending

Check if the GPU node is available:

```bash
kubectl describe pod -n ai open-webui-ollama-0
```

Common causes:

- GPU node not ready
- Insufficient GPU resources
- Missing node labels

### Certificate not issued

Check cert-manager logs:

```bash
kubectl logs -n cert-manager -l app=cert-manager
```

Verify the ClusterIssuer status:

```bash
kubectl describe clusterissuer letsencrypt-prod
```

### Ollama not responding

Check Ollama logs:

```bash
kubectl logs -n ai open-webui-ollama-0
```

### Models downloading slowly

Model downloads happen from Ollama's registry. Large models (13B+) can take 10-30 minutes depending on your connection.

## Next Steps

- [Configure OpenCode client](./opencode-client-setup.md) to connect your IDE to your private LLM
- Set up API keys in Open WebUI for programmatic access
- Explore the [Open WebUI documentation](https://docs.openwebui.com/) for advanced features like RAG and function calling
