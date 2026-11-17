# Enabling gVisor

This guide explains how to enable gVisor on your Gardener Kubernetes cluster. gVisor provides an additional layer of isolation between containerized workloads and the host kernel by intercepting application system calls in user space.

## Prerequisites

- A [Kubernetes cluster on Gardener](../creating-a-kubernetes-cluster.md) using `containerd` as the container runtime
- Access to the Gardener Dashboard (or permissions to edit the `Shoot` manifest)
- [kubectl](../gardenctl.md) configured to access your cluster

## What gVisor Provides

Enabling the gVisor OCI runtime on a Gardener worker pool automatically configures:

- The gVisor runtime binaries (`runsc` and `containerd-shim-runsc-v1`) on worker nodes
- `containerd` configuration to support sandboxed execution
- A `gvisor` Kubernetes `RuntimeClass` for scheduling sandboxed pods
- Automatic node labeling (`containerruntime.worker.gardener.cloud/gvisor=true`)

This allows you to run untrusted or multi-tenant pods in a user-space sandbox alongside standard `runc` workloads on the same cluster.

## Enabling gVisor via the Gardener Dashboard

1. Open your cluster in the **Gardener Dashboard** and navigate to the **Details**/**Workers** section.
2. Click the **Configure Workers** (settings) icon next to the **Worker** pools.
3. Expand the **Add. OCI Runtimes** dropdown menu.
4. Select **gvisor**.
5. Click **Save** to trigger a cluster reconciliation.

## Using gvisor

Specify `runtimeClassName: gvisor` in your workloads so as to use gvisor.

## Additional information
Feel free to check out the official [Gardener Gvisor extension repository](https://github.com/gardener/gardener-extension-runtime-gvisor) for further information.
