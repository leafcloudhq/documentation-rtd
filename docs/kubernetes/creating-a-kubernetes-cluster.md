# Creating a Kubernetes cluster

This tutorial will guide you through the process of creating a new Kubernetes cluster, using the OpenStack CLI. Container clusters are deployed and managed by the OpenStack Magnum project. You can use these clusters to deploy reproducible high-availability services and infrastructures 

To get started, you'll need to have a few things prepared: 

The OpenStack CLI: You can find a tutorial on how to set up the OpenStack CLI in the *Quick Start* section

Kubectl: This allows you to deploy your applications as a cluster. make sure it's the same version as used in the cluster you are deploying. You can download the kubectl here: 

`` 
https://v1-19.docs.kubernetes.io/docs/tasks/tools/install-kubectl/ 
`` 

## Configure the OpenStack CLI 

First, set up Python and a virtual environment. To do this run the following: 

`` 
python3 ~/virtualenv/openstack 

. ~/virtualenv/openstack/bin/activate 

pip install openstackclient python-magnumclient 
`` 

Check if you can reach the server list using OpenStack: 

`` 
openstack server list 
`` 
## Choose a template

 You can find a list of the various templates at ... Choose the one that best meets your needs.

## Create a Cluster 

`` 
openstack coe cluster create --cluster-template ${template_name} \ --master-count 1 \ 
--node-count ${node_count} \ 
--keypair ${keypair} \ 
${name} 
`` 

To check up on the installation enter the following: 

`` 
openstack coe cluster list 
`` 

When the status switches from CREATE_IN_PROGRESS to CREATE_COMPLETE the installation is complete and you can retrieve the config ($name is the name of your cluster: 

`` 
openstack coe cluster config $name 
`` 

This will download a file called 'config' to the working directory. 

`` 
export KUBECONFIG=/home/user/dir/config 
`` 

Now you need to open the Security Group rule, to communicate with the server.  

Go to Security Groups and select the Security Group of your master and click *Add Rule*. 

Open Port: Port 

Port: 6443 

CIDR: <your_IP>/32 

The config is now usable by kubectl and you should be able to reach your cluster by entering: 

`` 
kubectl get nodes -o wide 
``
 
## Configure Storage 

To configure Kubernetes for storage in OpenStack, first apply the storageclass yaml: 

`` 
kubectl apply -f ../magnum_additions/storageclass.yaml 
`` 

This makes Cinder available and set as the default storageclass. Label the nodes so they can access the volumes if sheduled. 

You can use the supplied script: 

`` 
./node.sh 
`` 