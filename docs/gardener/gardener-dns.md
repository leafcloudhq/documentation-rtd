# Using Gardener DNS with Ingress

This guide explains how to configure a new subdomain under the generated Gardener Shoot Domain using Kubernetes Ingress resources. 

## Prerequisites
- Access to the Gardener dashboard
- A running Kubernetes cluster managed by Gardener
- An existing Gardener Shoot Domain
- A configured `nginx` ingress controller
- A working `cert-manager` for SSL certificate management

## Finding Your Cluster's Domain
To locate the domain assigned to your cluster, navigate to the **Cluster Details** screen in the Gardener dashboard:

1. Open the Gardener dashboard.
2. Select your cluster from the list.
3. Navigate to **Infrastructure**.
4. Look for **Shoot Domain** – this is your cluster's base domain, which you can use to create subdomains.

For example, if your **Shoot Domain** is `my-awsome-cluster.a1b2c3.gardener.leaf.cloud`, you can create subdomains under it using an Ingress resource.

## Creating a New Subdomain
You can create a new subdomain under your Gardener Shoot Domain by defining an Ingress resource. The following example creates an Ingress for `custom-url.my-awsome-cluster.a1b2c3.gardener.leaf.cloud`:

### Example Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test
  namespace: test
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /
    dns.gardener.cloud/dnsnames: custom-url.my-awsome-cluster.a1b2c3.gardener.leaf.cloud
    dns.gardener.cloud/ttl: "600"
    dns.gardener.cloud/class: garden
spec:
  ingressClassName: nginx
  rules:
  - host: custom-url.my-awsome-cluster.a1b2c3.gardener.leaf.cloud
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
  tls:
  - hosts:
    - custom-url.my-awsome-cluster.a1b2c3.gardener.leaf.cloud
    secretName: test-ingress-tls

```

## Explanation of Key Fields

| Field                                      | Description |
|--------------------------------------------|-------------|
| `dns.gardener.cloud/dnsnames`             | Defines the DNS name that will be created within the Gardener-managed DNS zone. |
| `dns.gardener.cloud/ttl`                   | Sets the Time-to-Live (TTL) value for the DNS entry (e.g., `600` seconds). |
| `dns.gardener.cloud/class`                 | Specifies the DNS management class as `garden`. |
| `cert-manager.io/cluster-issuer`           | Uses Let's Encrypt for automatic TLS certificate provisioning. |
| `rules.host`                               | Defines the hostname for the Ingress resource. |
| `tls`                                      | Configures TLS for secure HTTPS access, with a corresponding `secretName` for the certificate. |




## Applying the Configuration
Save the above YAML configuration to a file (e.g., ingress.yaml) and apply it using:
kubectl apply -f ingress.yaml

## Verifying the DNS Record
Once the Ingress is applied, verify that the DNS record is created by checking:
dig custom-url.my-awsome-cluster.a1b2c3.gardener.leaf.cloud
You should see an A record resolving to the LoadBalancer IP of your nginx ingress controller.

!!! warning
  it can take several minutes before changes are propagated accross nameservers. To verify instant, try our nameserver ns1.leaf.cloud



## Conclusion

By applying this Ingress configuration, you have successfully created a subdomain under your Gardener-managed domain, allowing secure access to services deployed in your Kubernetes cluster.

