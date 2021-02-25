# Creating a PostgreSQL Cluster 

This tutorial will walk you through the steps to install and use the PostgreSQL Operator. This is used to manage highly available PostgreSQL clusters on Kubernetes

Before starting this tutorial, make sure you have completed all the steps in the *creating a kubernetes cluster* tutorial

## Install the PostgreSQL Operator 

First, we'll install the PostgreSQL Operator:

``` 
kubectl create namespace pgo 
kubectl apply -f https://raw.githubusercontent.com/CrunchyData/postgres-operator/v4.6.0/installers/kubectl/postgres-operator.yml 
``` 

## Install the pgo Client 

When the PostgreSQL Operator install is finished, run the client setup script: 

``` 
curl https://raw.githubusercontent.com/CrunchyData/postgres-operator/v4.6.0/installers/kubectl/client-setup.sh > client-setup.sh 
chmod +x client-setup.sh 
``` 

This will download the pgo client and provide instructions on how to use it in your environment. It will request the addition of certain variables, which will look something like: 

```
export PGOUSER="${HOME?}/.pgo/pgo/pgouser" 
export PGO_CA_CERT="${HOME?}/.pgo/pgo/client.crt" 
export PGO_CLIENT_CERT="${HOME?}/.pgo/pgo/client.crt" 
export PGO_CLIENT_KEY="${HOME?}/.pgo/pgo/client.key" 
export PGO_APISERVER_URL='https://127.0.0.1:8443' 
export PGO_NAMESPACE=pgo 
``` 

copy the variables as they are printed in ypur terminal and put them in your ~/.bashrc file on linux or in your ~/.bash_profile on mac and run source ~/.bashrc or ~/.bash_profile respectively

## Post-Installation Setup 

The PostgreSQL Operator is installed to the pgo namespace. First, check that the Kubernetes Deployment of the Operator exists and is healthy:

``` 
kubectl -n pgo get deployments 
``` 

Next, make sure the Pods that run the Operator are operational: 

``` 
kubectl -n pgo get pods 
``` 

To communicate with the PostgreSQL Operator API server, you need to set up a port forward to your local environment. You can do this by opening a new terminal window and running the following: 

``` 
kubectl -n pgo port-forward svc/postgres-operator 8443:8443 
``` 

Now go back to your original terminal window and verify your connection to the PostgreSQL Operator: 

``` 
pgo version 
``` 

## Creating a PostgreSQL Cluster 

This type of istall creates a pgo namespace, from which the PostgreSQL Operator manages PostgreSQL clusters. We'll start by creating a PostgreSQL cluster called leafdb: 

``` 
pgo create cluster -n pgo leafdb 
``` 

It may take a while for the cluster to be provisioned. You can see the cluster's status by using the pgo test command: 

``` 
pgo test -n pgo leafdb 
``` 

This test command gives you the basic information to connect to your PostgreSQL cluster from inside your Kubernetes environment. For more detailed information, enter the following: 

``` 
pgo show cluster -n pgo leafdb 
``` 

You have now created a PostgreSQL cluster. For more information on how to use the PostgreSQL Operator visit `https://postgres-operator.readthedocs.io/en/latest/`

 
 

 