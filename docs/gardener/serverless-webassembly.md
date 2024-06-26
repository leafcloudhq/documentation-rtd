# Harness the Potential of WebAssembly in Your Gardener Kubernetes Clusters
WebAssembly, often referred to simply as Wasm, is revolutionizing the way applications are developed and deployed. Offering significant benefits over traditional containerization methods like Docker, Wasm is faster, more efficient, and highly portable. It provides near-native performance, lightweight execution, and robust security, making it an incredibly effective and futuristic alternative.

## Getting Started with WebAssembly on Kubernetes

### Step-by-Step Installation Guide

To integrate WebAssembly into your Gardener Kubernetes clusters, follow these installation steps. This setup prepares your environment for running WebAssembly workloads efficiently on your nodes.
1. **Install KWasm Node Installer**: This step installs the KWasm node installer, preparing your nodes to run WebAssembly workloads. It configures the node's operating system and resources to support the WebAssembly runtime, enabling smooth deployment and execution of Wasm applications.
```bash
    kubectl apply -f https://raw.githubusercontent.com/KWasm/kwasm-node-installer/main/example/daemonset.yaml
```
2. **Install Cert-Manager**: Cert-Manager handles the certificates required for the Spin Operator, ensuring secure communication within your Kubernetes cluster.
```bash
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.3/cert-manager.yaml
```

3. **Apply Spin Operator Runtime Class**: This configures the runtime class for the Spin Operator, allowing it to manage and execute Wasm modules efficiently.
```bash
    kubectl apply -f https://github.com/spinkube/spin-operator/releases/download/v0.2.0/spin-operator.runtime-class.yaml
```

4. **Install Spin Operator Custom Resource Definitions (CRDs)**: CRDs define the custom resources that the Spin Operator will manage, enabling Kubernetes to understand and work with these new resource types.
```bash
    kubectl apply -f https://github.com/spinkube/spin-operator/releases/download/v0.2.0/spin-operator.crds.yaml
```

5. **Install Spin Operator with Helm**: Use Helm to install the Spin Operator, which orchestrates the deployment and management of Wasm modules in your cluster.
```bash
    helm install spin-operator \
        --namespace spin-operator \
        --create-namespace \
        --version 0.2.0 \
        --wait \
        oci://ghcr.io/spinkube/charts/spin-operator
```

6. **Apply Spin Operator Shim Executor**: Finally, set up the shim executor to enable the execution of Wasm modules, ensuring they run smoothly within your Kubernetes environment.
```bash
    kubectl apply -f https://github.com/spinkube/spin-operator/releases/download/v0.2.0/spin-operator.shim-executor.yaml
```


The installation part is now done. Let's test the setup by running a small sample Wasm application in your cluster:

```yaml
apiVersion: core.spinoperator.dev/v1alpha1
kind: SpinApp
metadata:
  name: simple-spinapp
spec:
  image: "ghcr.io/spinkube/containerd-shim-spin/examples/spin-rust-hello:v0.13.0"
  replicas: 1
  executor: containerd-shim-spin
```

You can then verify the deployment by checking the status of the SpinApp and the pods:
```bash
kubectl get spinapp
kubectl get pods
```

When you change the amount of replicas from 1 to 10 you will notice how quickly the additional replicas are deployed, showcasing the efficiency and speed of WebAssembly in handling scalable workloads.