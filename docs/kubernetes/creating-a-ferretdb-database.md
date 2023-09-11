# Creating a FerretDB database 

FerretDB is an Open Source alternative to MongoDB, built on Postgres. It serves as a stateless proxy, converting MongoDB protocol queries into SQL. It utilizes PostgreSQL as its underlying database engine.

FerretDB can be a great alternative to MongoDB if you prefer to use Open Source products.

Before starting this tutorial, make sure you have completed all the steps in the *creating-a-postgresql-cluster* tutorial.


## Create database for FerretDB

First, let's check if the hippo cluster form the *creating-a-postgresql-cluster* tutorial is up and running


``` 
kubectl -n postgres-operator describe postgresclusters.postgres-operator.crunchydata.com hippo

``` 

With our PostgreSQL instance properly set up, we can now move on to installing FerretDB.



Now we need to configure PostgreSQL according to FerretDB requirements. To do that, we're going to create a new user and password credential, and then create a database assigned with all privileges to the created user.

Let's create a Secret for a generated random password that will serve as the user credential for the PostgreSQL user and store it in a file called secret.sh:

```
#!/bin/sh

PASSWORD=$(openssl rand -base64 8)

kubectl -n postgres-operator create secret generic ferretdb-secret --from-literal=password=$PASSWORD
```

Run the following command to execute the script:


```
chmod +x secret.sh
./secret.sh

```


To create the user and database, we need to execute the psql command inside the pod since we have it running in Kubernetes. Here's how to do it:


```
PASSWORD=$(kubectl -n postgres-operator get secret ferretdb-secret -o=jsonpath='{.data.password}' | base64 -d)
POD_NAME=$(kubectl -n postgres-operator get pods --no-headers -o custom-columns=:metadata.name | grep '^hippo-instance' | head -1)
kubectl -n postgres-operator exec -it $POD_NAME -- psql -U postgres -c "CREATE USER ferretuser WITH PASSWORD '$PASSWORD';"

```

Next, we'll create a database named ferretdb and assign all privileges to the user (ferretuser) we created:

```
POD_NAME=$(kubectl -n postgres-operator get pods --no-headers -o custom-columns=:metadata.name | grep '^hippo-instance' | head -1)
kubectl -n postgres-operator exec -it $POD_NAME -- psql -U postgres -c "CREATE DATABASE ferretdb OWNER ferretuser;"

```


Check if the user and database were created successfully:

```
POD_NAME=$(kubectl -n postgres-operator get pods --no-headers -o custom-columns=:metadata.name | grep '^hippo-instance' | head -1)
kubectl -n postgres-operator exec -it $POD_NAME -- psql -U postgres -l

```

Great! Now we have a PostgreSQL instance running in Kubernetes with a user and database that we can use for FerretDB.



## Install FerretDB

Now that we have verified that PostgreSQL is working properly, it's time to set up and install FerretDB to communicate with our PostgreSQL cluster. To do this, we'll need to create and apply a deployment and service manifest. This YAML file will define the container specifications for FerretDB, any associated MongoDB components, and the necessary service configuration to establish a connection to PostgreSQL. We've also included a Secret for the ferretuser password we created earlier.

The following deployment YAML defines our FerretDB instance, allowing it to communicate with our PostgreSQL setup. It specifies the container configurations, connection details, and the service through which external requests can connect.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ferretdb
  namespace: postgres-operator
  labels:
    app: ferretdb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ferretdb
  template:
    metadata:
      labels:
        app: ferretdb
    spec:
      containers:
        - name: ferretdb
          image: ghcr.io/ferretdb/ferretdb
          ports:
            - containerPort: 27017
          env:
            - name: POSTGRES_USERNAME
              value: ferretuser
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ferretdb-secret
                  key: password
            - name: FERRETDB_POSTGRESQL_URL
              value: postgres://$(POSTGRES_USERNAME):$(POSTGRES_PASSWORD)@hippo-ha:5432/ferretdb
---
apiVersion: v1
kind: Service
metadata:
  name: ferretdb-service
  namespace: postgres-operator
spec:
  selector:
    app: ferretdb
  ports:
    - name: mongo
      protocol: TCP
      port: 27017
      targetPort: 27017
```


Apply the deployment YAML:

```
kubectl apply -f deployment.yaml

```

Check if all pods are starting up without issues:


```
kubectl get pods -n postgres-operator

```


Next you need to connect using your FerretDB URI, where username and password should correspond with the PosgreSQL credentials set earlier, and you can get the FERRETDB SVC by running:

```
kubectl get svc -n postgres-operator

```

Retrieve your password:

```
kubectl get secret ferretdb-secret -n postgres-operator -o jsonpath='{.data.password}' | base64 --decode
```


To open a mongosh shell run the following command:

```
kubectl -n postgres-operator run mongosh --image=rtsp/mongosh --rm -it -- bash
```

Once the mongosh is open, connect to your FerretDB instance using the command:

```
mongosh "mongodb://ferretuser:<password>@{FERRETDB SVC}/ferretdb?authMechanism=PLAIN"
```


And that's it!. You've connected to FerretDB.
For more details on how to use the PostgreSQL Operator, visit: [FerretDB](https://www.ferretdb.io/)