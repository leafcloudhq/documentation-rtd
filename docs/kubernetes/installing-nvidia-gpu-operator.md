# Installing the NVIDIA GPU Operator

This guide explains how to install the NVIDIA GPU Operator on your Gardener Kubernetes cluster. The GPU Operator automates the management of all NVIDIA software components needed to provision and manage GPUs in Kubernetes.

## Prerequisites

- A [Kubernetes cluster on Gardener](../gardener/creating-a-kubernetes-cluster.md) with GPU-enabled worker nodes (e.g., V100, A30, A100)
- [Helm](./installing-helm.md) installed on your local machine
- [kubectl](../gardener/gardenctl.md) configured to access your cluster

## What the GPU Operator Provides

The NVIDIA GPU Operator automatically installs and manages:

- NVIDIA drivers
- NVIDIA Container Toolkit
- Kubernetes device plugin for GPUs
- GPU monitoring tools

This means you don't need to manually install drivers on your nodes.

## Installation

Add the NVIDIA Helm repository:

```bash
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
```

Create the namespace and install the operator:

```bash
kubectl create ns gpu-operator
kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged
helm install gpu-operator nvidia/gpu-operator \
  --namespace gpu-operator \
  --set toolkit.enabled=true \
  --version v24.9.0 \
  --wait
```

## Verifying the Installation

Wait for all pods to be ready (this can take a few minutes):

```bash
kubectl get pods -n gpu-operator -w
```

All pods should eventually show `Running` or `Completed` status.

Verify the GPU is detected by checking the node labels:

```bash
kubectl describe nodes | grep -A5 "nvidia.com/gpu"
```

You should see output similar to:

```
nvidia.com/gpu.count=1
nvidia.com/gpu.present=true
nvidia.com/gpu.product=NVIDIA-A30
```

You can also check the allocatable resources:

```bash
kubectl get nodes -o json | jq '.items[].status.allocatable | select(.["nvidia.com/gpu"] != null)'
```

## Using GPUs in Your Pods

Once the operator is installed, you can request GPUs in your pod specifications:

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

To ensure your pods are scheduled on GPU nodes, use a node selector:

```yaml
nodeSelector:
  nvidia.com/gpu.present: "true"
```

## Troubleshooting

### Pods stuck in Pending

Check if the GPU node is available and has allocatable GPUs:

```bash
kubectl describe node <node-name> | grep -A10 "Allocatable"
```

### Driver installation failing

Check the driver installer pod logs:

```bash
kubectl logs -n gpu-operator -l app=nvidia-driver-daemonset
```

### GPU not detected

Verify the GPU hardware is present on the node:

```bash
kubectl debug node/<node-name> -it --image=ubuntu -- lspci | grep -i nvidia
```

## Upgrading

To upgrade the GPU Operator to a newer version:

```bash
helm repo update
helm upgrade gpu-operator nvidia/gpu-operator \
  --namespace gpu-operator \
  --set toolkit.enabled=true \
  --version <new-version>
```

## Uninstalling

To remove the GPU Operator:

```bash
helm uninstall gpu-operator -n gpu-operator
kubectl delete ns gpu-operator
```
