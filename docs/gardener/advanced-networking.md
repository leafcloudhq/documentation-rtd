# Advanced Network Options in Gardener

When you create a cluster in Gardener, Gardener will provision the necessary resources in OpenStack, such as instances and networks. In most cases, the default settings are sufficient. However, there are situations where you may need more control over these configurations.

In this tutorial, we will explore two advanced network options, both of which can only be applied during cluster creation:

1. **Changing the IP Range of Your Pods**: This can be useful in a multi-cluster setup to prevent overlapping IP ranges.

2. **Using an Existing OpenStack Router in Your Kubernetes Cluster**: This is beneficial if you have IP whitelisting with external parties. It allows you to recreate your cluster while retaining the same router's external IP.

By following this tutorial, you will learn how to implement these advanced network configurations in Gardener.

## Changing the IP Range of Your Pods

To change the IP range of your pods during cluster creation in Gardener, follow these steps:

1. **Create a Cluster in the Gardener Dashboard**:
    - Go to the Gardener dashboard and start the cluster creation process.
    - Configure the settings and other options as desired.
    - Instead of creating the cluster immediately, copy the YAML configuration from the YAML tab.

2. **Modify the YAML Configuration**:
    - Add the following snippet to the networking section of the YAML configuration to specify the IP range for your pods:
```yaml
networking:
pods: 192.168.0.0/22
```

3. **Target Your Garden with the `gardenctl` Command**:
    - Open your terminal and run the following command to target your garden:
```bash
gardenctl target --garden leafcloud-production --project <your-project-id>
```

4. **Apply Your YAML Configuration**:
    - Apply the modified YAML configuration to create your shoot cluster with the specified pod IP range:
```bash
gardenctl apply -f your-modified-cluster-config.yaml
```

By following these steps, you can change the IP range of your pods during the cluster creation process in Gardener.

## Using an Existing OpenStack Router in Your Kubernetes Cluster

To use an existing OpenStack router during cluster creation in Gardener, follow these steps:

1. **Login to OpenStack and Create a Router**:
    - Login to your OpenStack dashboard.
    - Create a router and connect it to the external network.
    - Copy the ID of the created router.

2. **Create a Cluster in the Gardener Dashboard**:
    - Go to the Gardener dashboard and start the cluster creation process.
    - Configure the settings and other options as desired.
    - Instead of creating the cluster immediately, copy the YAML configuration from the YAML tab.

3. **Modify the YAML Configuration**:
    - Add the following snippet to the `provider` section of the YAML configuration to specify the router ID:
```yaml
provider:
type: openstack
infrastructureConfig:
    apiVersion: openstack.provider.extensions.gardener.cloud/v1alpha1
    kind: InfrastructureConfig
    networks:
    workers: 10.250.0.0/16
    router:
        id: 43f4ed65-5486-4f00-a392-79f9e28f6e36
```

4. **Target Your Garden with the `gardenctl` Command**:
    - Open your terminal and run the following command to target your garden:
```bash
gardenctl target --garden leafcloud-production --project <your-project-id>
```

5. **Apply Your YAML Configuration**:
    - Apply the modified YAML configuration to create your shoot cluster with the specified router:
```bash
gardenctl apply -f your-modified-cluster-config.yaml
```

By following these steps, you can use an existing OpenStack router during the cluster creation process in Gardener.
