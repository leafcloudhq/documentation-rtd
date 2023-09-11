# Creating a PostgreSQL Cluster 

This tutorial will walk you through the steps to install and use the PostgreSQL Operator. This is used to manage highly available PostgreSQL clusters on Kubernetes

Before starting this tutorial, make sure you have completed all the steps in the *creating a kubernetes cluster* tutorial

## Install the PostgreSQL Operator 

First, we'll install the PostgreSQL Operator:

clone the following repository:

```
git clone --depth 1 https://github.com/CrunchyData/postgres-operator-examples

```

This will create a namespace called postgres-operator and create all of the objects required to deploy PostgreSQL Operator.

```
kubectl apply -k kustomize/install/namespace
kubectl apply --server-side -k kustomize/install/default

``` 


To check on the status of your installation, you can run the following command:

```
kubectl -n postgres-operator get pods \
  --selector=postgres-operator.crunchydata.com/control-plane=postgres-operator \
  --field-selector=status.phase=Running

```

Wait unitl pod is in the running status.

Before creating the PostgreSQL cluster, we need to modify the ~/kustomize/postgres/postgres.yaml file from the cloned folder, and enable the cluster to use a data volume claim that specifies the 'cinder-csi' storage class and a capacity request of 1Gi. It should look like this:

``` 
apiVersion: postgres-operator.crunchydata.com/v1beta1
kind: PostgresCluster
metadata:
  name: hippo
spec:
  image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-15.3-2
  postgresVersion: 15
  instances:
    - name: instance1
      dataVolumeClaimSpec:
        accessModes:
          - 'ReadWriteOnce'
        resources:
          requests:
            storage: 1Gi
        storageClassName: cinder-csi
  backups:
    pgbackrest:
      image: registry.developers.crunchydata.com/crunchydata/crunchy-pgbackrest:ubi8-2.45-2
      repos:
        - name: repo1
          volume:
            volumeClaimSpec:
              accessModes:
                - 'ReadWriteOnce'
              resources:
                requests:
                  storage: 1Gi
              storageClassName: cinder-csi
``` 


Now let's create the PostgreSQL cluster by running the following command:

``` 
kubectl apply -k kustomize/postgres

``` 

You can track the progress of your cluster using the following command:

```
kubectl -n postgres-operator describe postgresclusters.postgres-operator.crunchydata.com hippo
```


As part of creating a Postgres cluster, the Postgres Operator creates a PostgreSQL user account. The credentials for this account are stored in a Secret that has the name <clusterName>-pguser-<userName>.


Retrieve the user's credentials:

For username:

```
(kubectl -n postgres-operator get secrets hippo-pguser-hippo -o go-template='{{.data.password | base64decode}}')
```

For password:

```
(kubectl -n postgres-operator get secrets hippo-pguser-hippo -o go-template='{{.data.password | base64decode}}')
```
 

Test the connection by port-forwarding your localhost to the PostgreSQL Operator:


 ```
PG_CLUSTER_PRIMARY_POD=$(kubectl get pod -n postgres-operator -o name \
  -l postgres-operator.crunchydata.com/cluster=hippo,postgres-operator.crunchydata.com/role=master)
kubectl -n postgres-operator port-forward "${PG_CLUSTER_PRIMARY_POD}" 5432:5432
```


Now, you can try to login to your PostgreSQL installation on localhost:5432.

Congratulations! You have now created a PostgreSQL cluster. For more details on how to use the PostgreSQL Operator, visit `https://postgres-operator.readthedocs.io/en/latest/`