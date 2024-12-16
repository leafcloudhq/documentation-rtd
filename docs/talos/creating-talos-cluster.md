# Talos Linux Kubernetes cluster on LeafCloud

Talos is a modern operating system purpose-built for Kubernetes clusters. Its immutable and minimal design eliminate configuration drift and reduce your cluster's attack surface.  All components communicate through mTLS out of the box and are centrally managed via an API.

Are you looking to run your workloads on Kubernetes but don't want to manage the control plane yourself? Our [Gardener-powered Kubernetes service](https://leaf.cloud/products/kubernetes) offers a fully managed, integrated Kubernetes cluster running in your LeafCloud account. You can focus on developing and running your applications while we handle the maintenance and scaling!

In this tutorial we'll show you how to create a ready-to-use Talos Linux based Kubernetes cluster in your LeafCloud account.

## Prerequisites

Before we begin you need both the Openstack command line set up and configured (see [Getting Started -> How to use the Openstack CLI](Getting-Started/Using-Openstack-CLI)) and _talosctl_ set up on your local machine (see [the talosctl installation instructions](https://www.talos.dev/v1.8/talos-guides/install/talosctl/)).


## 1. Network Environment

Talos consists of self-contained control plane and worker nodes. Before we can deploy those, we'll need an Openstack network to deploy them in. 

In this tutorial, our nodes will be running in `10.22.0.0/24` whereas the Kubernetes pods will run in `10.244.0.0/16` and the services in `10.96.0.0/12` by default.  

```sh
openstack network create talos-network
openstack subnet create talos-subnet \
    --network talos-network \
    --subnet-range 10.22.0.0/24 \
    --dns-nameserver 8.8.8.8
```

To allow our nodes to communicate with each other, we create a security group allowing all TCP traffic inside of the network.  Later these security groups will be attached to the virtual machines we'll be creating.
```sh
openstack security group create talos-sg
openstack security group rule create talos-sg --protocol tcp
```

Because these are private networks, we're creating a load balancer with a public IP to make the Talos and Kubernetes APIs publicly available. The load balancer goes in font our three control nodes so that the Talos and Kubernetes Control APIs remain available even when one control plane node goes down.

!!! note
    The `openstack loadbalancer` commands requires the octavia client which can be installed with `pip install python-octaviaclient`.
```sh
openstack loadbalancer create --name talos-control-plane --vip-subnet-id $(openstack subnet show talos-subnet -f value -c id); sleep 60
openstack loadbalancer listener create --name talos-control-plane-listener --protocol TCP --protocol-port 50000 talos-control-plane; sleep 3
openstack loadbalancer listener create --name talos-k8s-api-listener --protocol TCP --protocol-port 6443 talos-control-plane; sleep 3
openstack loadbalancer pool create --name talos-control-plane-pool --listener talos-control-plane-listener --protocol TCP --lb-algorithm ROUND_ROBIN; sleep 3
openstack loadbalancer pool create --name talos-k8s-api-pool --listener talos-k8s-api-listener --protocol TCP --lb-algorithm ROUND_ROBIN; sleep 3
openstack loadbalancer healthmonitor create --delay 10 --max-retries 3 --timeout 15 --type TCP talos-control-plane-pool; sleep 3
openstack loadbalancer healthmonitor create --delay 10 --max-retries 3 --timeout 15 --type TCP talos-k8s-api-pool; sleep 3
```

```sh
openstack router create talos-router
openstack router set talos-router --external-gateway external
openstack router add subnet talos-router talos-subnet

PUBLIC_IP=$(openstack floating ip create external | grep -oP '(?<=floating_ip_address \| )\S+')
echo "PUBLIC_IP: $PUBLIC_IP"
```

Hold on to the `$PUBLIC_IP`, we will need it in our next step! 

<figure markdown="span">
    <img alt="Talos Network Topology" src="../../images/INF-50_Talos_network_topo.png" />
    <figcaption>our network topology: the private telos-net network connected to the external world through a router</figcaption>
</figure>




## 2. Generate Talos Config Files

Before we can deploy our machines, we'll need to generate the configuration which fed directly into the VMs during the creation step. As Talos speaks mTLS, we'll need to configure it for the public endpoint through which we'll be accessing the control plane.  Because our setup is behind a load balancer, we'll configure talos to accept certificates generated for the public IP.  

```sh
echo -e "machine:\n  certSANs:\n    - ${PUBLIC_IP}" > patch.yaml
talosctl gen config talos-k8s-openstack-tutorial https://${PUBLIC_IP}:6443 --config-patch @patch.yaml
```

This will generate the following files in the current working directory: `controlplane.yaml`, `worker.yaml`, and `talosconfig`. We encourage you to explore these and see all the configuration possibilities Talos offers!

## 3. Deploying Compute Resources

Now that our network is ready we can deploy the control plane and worker nodes into it. Configuration is provided by the yaml files we generated in the previous step, and is fed straight into the VMs. 

We will be running three control plane nodes for high availability. To bootstrap the control plane, we'll first assign our public IP to a single machine, instruct it to bootstrap, then attach that public IP to our load balancer.
```sh
for i in 1 2 3; do
  openstack server create control-plane-$i \
    --flavor ec1.small \
    --image talos-1.8.1 \
    --nic net-id=$(openstack network show talos-network -f value -c id) \
    --security-group talos-sg \
    --user-data controlplane.yaml
done
```
Now that we have our control plane, we can add it behind the load balancer.
```sh
for i in 1 2 3; do
  MEMBER_IP=$(openstack port list --server control-plane-$i -f value -c "Fixed IP Addresses" | grep -oP '10.22.0.\d+')
  openstack loadbalancer member create --subnet-id $(openstack subnet show talos-subnet -f value -c id) \
    --address $MEMBER_IP --protocol-port 6443 talos-k8s-api-pool; sleep 3
  openstack loadbalancer member create --subnet-id $(openstack subnet show talos-subnet -f value -c id) \
    --address $MEMBER_IP --protocol-port 50000 talos-control-plane-pool; sleep 3

done

openstack floating ip set $PUBLIC_IP --port $(openstack loadbalancer show talos-control-plane -c vip_port_id -f value)
```


Then initialize the control plane! All three machines should discover each other and form a cluster. This will take a while and some error messages (such as about timeouts) are to be expected.
```sh
talosctl --talosconfig talosconfig bootstrap -e $PUBLIC_IP -n $PUBLIC_IP
```
<figure markdown="span">
    <img alt="Talos Control Plane Node Bootstrapped" src="../../images/INF-50_Talos_controlplane.png" />
    <figcaption>the console of a bootstrapped control plane node</figcaption>
</figure>

Let's verify our cluster!

So far we only have a control plane.  We need a worker node to run pods on: 
```sh
openstack server create worker-1 \
  --flavor ec1.xsmall \
  --image talos-1.8.1 \
  --nic net-id=$(openstack network show talos-network -f value -c id) \
  --security-group talos-sg \
  --user-data worker.yaml
```

Using `talosctl`, we can get the configuration needed to access the cluster: 
```sh
talosctl --talosconfig talosconfig -e $PUBLIC_IP -n $PUBLIC_IP kubeconfig ./kubeconfig
export KUBECONFIG=./kubeconfig
kubectl get nodes -o wide
```

Deploy a pod on our new Kubernetes cluster:
```sh
kubectl run nginx --image=nginx 
kubectl get pods -o wide
```

## Drawing the rest of the Owl

You should now have a robust, self-hosted Kubernetes cluster with a highly available control plane - a solid start! 

The value of Kubernetes clusters is not just in hosting workloads but also in providing abstractions that allow for auto-scaling, self-healing systems. These features can be provided by Openstack, for which there are actively maintained controllers available:

- https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/octavia-ingress-controller/using-octavia-ingress-controller.md
- https://github.com/kubernetes/cloud-provider-openstack/tree/master?tab=readme-ov-file 
