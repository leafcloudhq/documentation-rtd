# Temporary Kubeconfig

From Kubernetes 1.27 onwards, Gardener enforces increased security by disallowing static kubeconfig files. Instead, you can generate temporary kubeconfig files with an expiration date. This ensures secure access to your shoot cluster with cluster-admin privileges for a limited time, eliminating the need to store long-term credentials.

## What is a Temporary Kubeconfig?

A temporary kubeconfig allows you to access your shoot cluster with cluster-admin privileges, but only for a short period. The credentials expire after a set time, requiring renewal for continued access. This method ensures secure access without the risk associated with storing long-term credentials.

## How to Generate a Temporary Kubeconfig

Follow these steps to generate and use a temporary kubeconfig:

1. **Target Your Garden**: Use `gardenctl`  Use gardenctl to set your current garden and project.
```bash
gardenctl target --garden your-garden --project project-id
```

2. **Set Environment Variables**: Define your namespace (optional if you already know it) and shoot name.

(Optional) Find the namespace of your shoot:
```bash
kubectl get shoots -o json | jq -r '.items[0].metadata.namespace'
```

```bash
export NAMESPACE=garden-my-namespace
export SHOOT_NAME=my-shoot
```


3. **Create the Temporary Kubeconfig**:Define your namespace (optional if you already know it) and shoot name.

```bash

kubectl create \
    -f <(printf '{"spec":{"expirationSeconds":600}}') \
    --raw /apis/core.gardener.cloud/v1beta1/namespaces/${NAMESPACE}/shoots/${SHOOT_NAME}/adminkubeconfig | \
    jq -r ".status.kubeconfig" | \
    base64 -d > kubeconfig.yaml
```

Use the Temporary Kubeconfig: Use the generated kubeconfig to interact with your shoot cluster.

```bash

    export KUBECONFIG=$(pwd)/kubeconfig.yaml
    kubectl get pods
```















