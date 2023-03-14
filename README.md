![wikijs](https://user-images.githubusercontent.com/50842626/207669111-93310e36-fdc5-4f2f-a32c-d1f4a3a4726a.png)

## Install and Configure Wiki.js on Kubernetes Cluster

Install and configure Wiki.js on Kubernetes Cluster on your system with the aid of the below steps.

### Step 1 – Create Namespace

Normally, a namespace is used to partition a single Kubernetes cluster into many virtual clusters. Begin by creating the namespace for wiki.js as below.

```
kubectl create namespace wikijs
```

Verify the namespace was created

```
kubectl get namespaces
```
### Step 2 – Create the Secrets file

The secret file contains the username and passwords to be created for the below database.

Generate your own credentials for the **`DB_NAME`**, **`DB_USER`** and **`DB_PASS`** variables. See below examples:

```
# Wiki.js PostgreSQL database name
$ echo -n 'wikijsdb' | base64
d2lraWpzZGI=

# Wiki.js PostgreSQL user
$ echo -n 'wikijs' | base64
d2lraWpz

# Wiki.js PostgreSQL user Password
$ echo -n 'WikijsUserPassw0rd' | base64
V2lraWpzVXNlclBhc3N3MHJk
```

The fields **`DB_NAME`**, **`DB_USER`** and **`DB_PASS`** are your secrets, and you can change the values according to your preference. 

Now create a secret file as below.

```
vim wikijs-secret.yaml
```

In the file, add the below lines replacing appropriately.

```
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: wikijs
type: Opaque
data:
  DATABASE_NAME: d2lraWpzZGI= # is used to create a database to be used as the default.
  DATABASE_USER: d2lraWpz # is used to create a superuser with the name set in the environment variable.
  DATABASE_PASSWORD: V2lraWpzVXNlclBhc3N3MHJk # is used to set the superuser password
```

Apply the made changes.

```
kubectl apply -f wikijs-secret.yaml
```

Verify if your change is made.

```
kubectl get secret -n wikijs
```
### Step 3 – Create the ConfigMap file

Create the configMap file which contains the wiki.js configurations details. 

You might also need to change the value of these variables **`DB_HOST`**, **`DB_PORT`**, and **`DB_TYPE`**  in the below YAML file.

Here i’m using database type as postgres, you can your own database type from the following **`DB_TYPE`**: Type of database (*mysql, postgres, mariadb, mssql or sqlite*).

Now create a ConfigMap file as below.

```
vim wikijs-configmap.yaml
```

In the file, add the below lines replacing appropriately.

```
apiVersion: v1
data:
  DB_HOST: "postgres.wikijs.svc.cluster.local"
  DB_PORT: "5432"
  DB_TYPE: "postgres"
  HA_ACTIVE: "true"
kind: ConfigMap
metadata:
  name: postgres-config
  namespace: wikijs
```

Apply the made changes.

```
kubectl apply -f wikijs-configmap.yaml
```

Verify if your change is made.

```
kubectl get configmaps -n wikijs
```
### Step 4 – Create the Database Pod for wiki.js

This config file contains the database details for **Wiki.js**. In this guide, we will use the PostgreSQL database which can be configured as below.

#### Create and Apply Persistent Storage Volume and Persistent Volume Claim

In order to ensure data persistence, you should use a persistent volume (PV) and persistent volume claims (PVC). A persistent volume (PV) is a durable volume that will remain even if the pod is deleted and stores data.

A persistent volume claim (PVC) is how users request and consume PV resources. Think of it as requesting the PV with parameters such as size of your storage disk, access modes, and storage class.

To deploy stateful applications such as a PostgreSQL database, for example, you’ll need to create a PVC for the database data. You can create a pod that mounts the PVC and runs the MySQL database.

For this tutorial, you will move forward with a local volume, using **`/mnt/data`** as the path to volume:

```
sudo mkdir /mnt/data
```

Create the postgres-pvc-pv.yaml file as below.

```
vim postgres-pvc-pv.yaml
```

In the file, add the below lines. Here do not alter anything.

```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: postgres-pv-volume  # Sets PV's name
  namespace: wikijs
  labels:
    type: local  # Sets PV's type to local
    app: postgres
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi # Sets PV Volume
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/data"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim  # Sets name of PVC
  namespace: wikijs
  labels:
    app: postgres
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany  # Sets read and write access
  resources:
    requests:
      storage: 5Gi  # Sets volume size
```

Run the following command to create a new PVC and PV for your PostgreSQL deployment:

```
kubectl apply -f postgres-pvc-pv.yaml 
```

Use the command below to check if PVC is bound to PV:

```
kubectl get pvc
```

If the **`STATUS`** is “Bound”, you can use it for your deployments.

#### Create and Apply PostgreSQL Deployment

Deployments are a way to manage rolling out and updating applications in a Kubernetes cluster. They provide a declarative way to define how an application should be deployed and updated, and can be used to roll back to previous versions if needed.

After creating PVCs, PVs, and Secrets, you can create a stateful application by creating a stateful pod as follows:

```
vim postgres-deployment.yaml
```

In the file, add the below lines. Here do not alter anything. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres  # Sets Deployment name
  namespace: wikijs
spec:
  replicas: 1
  selector:
    matchLabels:
      service: postgres
      app: postgres
  template:
    metadata:
      labels:
        service: postgres
        app: postgres
    spec:
      volumes:
        - name: postgredb
          persistentVolumeClaim:
            claimName: postgres-pv-claim
      containers:
        - name: postgres
          image: postgres:10.1 # Sets Image
          imagePullPolicy: "IfNotPresent"
          resources:
            limits:
              cpu: 500m
              memory: 512Mi
            requests:
              cpu: 250m
              memory: 256Mi
          ports:
            - containerPort: 5432  # Exposes container port
          env:
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: DATABASE_NAME
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                 name: postgres-secret
                 key: DATABASE_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                 name: postgres-secret
                 key: DATABASE_PASSWORD
            - name: TZ
              value: "America/Recife"
            - name: PGTZ
              value: "America/Recife"
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgredb
```

The following command will create a new PostgreSQL deployment:

```
kubectl apply -f postgres-deployment.yaml
```

#### Create and Apply PostgreSQL Service

Kubernetes services help you expose ports in various ways, including through a NodePort. NodePorts expose a service on every node in a cluster, meaning that the service is accessible from outside the cluster. This can be useful for services that need to be accessible from outside the cluster. To keep things simple for this tutorial, you’ll expose the database using NodePort with the help of the following manifest:

```
vim postgres-service.yaml
```

Now we will configure PostgreSQL Service.

```
apiVersion: v1
kind: Service
metadata:
  name: postgres # Sets service name
  namespace: wikijs
  labels:
    app: postgres # Labels and Selectors
    service: postgres
spec:
  #type: NodePort # Sets service type
  selector:
    app: postgres
  ports:
  - name: postgres
    protocol: TCP
    port: 5432 
```
  
  The command below will create a new PostgreSQL service which helps you to connect to psql:

  ```
  vim kubectl apply -f postgres-service.yaml 
  ```

  kubectl exec -it **[pod-name]** --  psql -h localhost -U admin --password -p 5432 postgresdb

  #### Connect to PostgreSQL

The Kubernetes command line client ships with a feature that lets you connect to a pod directly from your host command line. The kubectl exec command accepts a pod name, any commands that should be executed, and an interactive flag that lets you launch a shell. You’ll use kubectl exec to connect to PostgreSQL pod:

```
kubectl exec -n [namespace] -it [pod-name] -- psql -h localhost -U wikijs --password -p 5432 wikijsdb
```

#### Connecting to PostgreSQL via PostgreSQL Client

If you prefer to connect to the PostgreSQL shell using the PostgreSQL client, run the psql command instead. Be sure the PostgreSQL client packages are installed on your local machine.

1. Run the psql command below to connect to the PostgreSQL pod through the ClusterIP with the port PostgreSQL.

```
psql -h 10.103.35.241 -U wikijs --password -p 5432 wikijsdb
```

2. Input the password for your database to connect to the PostgreSQL shell.

3. Finally, run the query below to get information about your connection and list databases.

```
# Checking PostgreSQL connection
\conninfo

# List databases
\l
```

### Step 5 – Deploy the Wiki.js Service and application

Here, we can deploy the service as a NodePort, ClusterIP, or load balancer. First, create the file

```
vim wikijs-service.yaml
```

Now we will configure Wiki.js.

Service Type as ClusterIP.

```
apiVersion: v1
kind: Service
metadata:
  name: "wikijs"
  namespace: wikijs
spec:
  type: ClusterIP
  selector:
    app: "wikijs"
  ports:
    - name: http
      port: 3000
```

Or, Service Type as LoadBalancer

```
apiVersion: v1
kind: Service
metadata:
  name: wikijs
  namespace: wikijs
spec:
  type: LoadBalancer
  selector:
    app: wikijs
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
```

Here i deployed the service Type as ClusterIP. If you want, you can deploy service Type as LoadBalancer.

The command below will create a Wikijs service

```
kubectl apply -f wikijs-service.yaml
```

Verify if your change is made.

```
kubectl get svc -n wikijs
```

#### Create and Apply Wiki.js Deployment

Now proceed to the wiki.js deployment.

```
vim wikijs-deployment.yaml
```

In the file, add the below lines, here don’t replace anything, we are simply mapping the above configs. I have set the imagePullPolicy parameter to Always in order to always get the latest Wiki.js release.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wikijs
  namespace: wikijs
  labels:
    app: wikijs
spec:
  selector:
    matchLabels:
      app: wikijs
  template:
    metadata:
      labels:
        app: wikijs
    spec:
      containers:
       - name: wikijs
         image: requarks/wiki:2
         #image: requarks/wiki:latest
         imagePullPolicy: "IfNotPresent"
         ports:
          - containerPort: 3000
            name: http
         resources:
            limits:
              cpu: 500m
              memory: 512Mi
            requests:
              cpu: 250m
              memory: 256Mi
         env:
            - name: HA_ACTIVE
              valueFrom:
                configMapKeyRef:
                  key: HA_ACTIVE
                  name: postgres-config
            - name: DB_TYPE
              valueFrom:
                configMapKeyRef:
                  key: DB_TYPE
                  name: postgres-config
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  key: DB_HOST
                  name: postgres-config
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  key: DB_PORT
                  name: postgres-config
            - name: DB_NAME
              valueFrom:
                secretKeyRef:
                 name: postgres-secret
                 key: DATABASE_NAME
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                 name: postgres-secret
                 key: DATABASE_USER
            - name: DB_PASS
              valueFrom:
                secretKeyRef:
                 name: postgres-secret
                 key: DATABASE_PASSWORD
      
```

Mapping the configMap and secret variables to the Deployment. Pulled the official wikijs docker image.

Now apply the made changes.

```
kubectl apply -f wikijs-deployment.yaml
```

After the deployment, check the status of the pod, if the status is Running then look into the logs and you should see something like this.

### Backup Postgres

Create a secret with postgres admin password. It needs to be in the format that can be read by pg_dump tool. You can find more info about pg_dump password file [here](https://www.postgresql.org/docs/9.1/libpq-pgpass.html).

If you don’t feel like opening the link above, here is the format of the file:

```
hostname:port:database:username:password
```

So for a postgres Depoyment/StatefulSet with database test_database that is exposed as a service postgres on port 5432, your file should look something like this:

```
postgres.wikijs.svc.cluster.local:5432:wikijsdb:wikijs:WikijsUserPassw0rd
```

Kubernetes secrets require data to be base64 encoded, so to create a secret value, invoke the command below in bash (you need to have base64 tool installed as well):

```
echo "postgres.wikijs.svc.cluster.local:5432:wikijsdb:wikijs:WikijsUserPassw0rd" | base64
```

The output will be:

```
cG9zdGdyZXMud2lraWpzLnN2Yy5jbHVzdGVyLmxvY2FsOjU0MzI6d2lraWpzZGI6d2lraWpzOldpa2lqc1VzZXJQYXNzdzByZA==
```

Create this is the value into your kubernetes secret:

```
apiVersion: v1
kind: Secret
metadata:
  name: pgpass
  namespace: wikijs
data:
  pgpass: cG9zdGdyZXMud2lraWpzLnN2Yy5jbHVzdGVyLmxvY2FsOjU0MzI6d2lraWpzZGI6d2lraWpzOldpa2lqc1VzZXJQYXNzdzByZA==
```

>**Note**
>If your postgres password contains special characters, such as exclamation mark or backslash, you need to escape them before you feed them into **base64** command. Best way to test if your password has been encoded correctly is to decode the base64 string and check the password for missing characters.

Example:

```
printf "%s" 'cdt_main!@#$' | base64
```

Create this is the value into your kubernetes CronJobs:

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: wikijs
spec:
  # Backup the database every day at 10AM
  schedule: "0 10 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: postgres-backup
            image: postgres:10.1
            command: ["/bin/sh"]
            args: ["-c", 'echo "$PGPASS" > /root/.pgpass && chmod 600 /root/.pgpass && pg_dump -U wikijs -h postgres.wikijs.svc.cluster.local wikijsdb > /var/backups/backup-$(date +"%m-%d-%Y-%H-%M").sql']
            env:
            - name: PGPASS
              valueFrom:
                secretKeyRef:
                  name: pgpass
                  key: pgpass
            volumeMounts:
            - mountPath: /var/backups
              name: postgres-storage
          restartPolicy: Never
          volumes:
          - name: postgres-storage
            hostPath:
            # Ensure the file directory is created.
              path: /var/volumes/postgres-backups
              type: DirectoryOrCreate
```    

Once you apply the cronjob to your cluster (must be in the same namespace as database), it will backup your database every day at 10AM.

### Restore from a Postgres Backup

Copy database backup into pod:

```
kubectl -n wikijs cp backup-02-23-2023-13-00.sql postgres-bcc5c45b8-lnhpl:/tmp/wikijs.sql
```

To manually do this, you should once again connect to you postgres pod in your shell. The first step is to drop your database. Ensure you have correctly followed the above steps to make sure you don't lose any data. To drop your database run:

```
psql -U wikijs template1 -c 'drop database wikijsdb;'
```

Now create your database again, with:

```
psql -U wikijs template1 -c 'create database wikijsdb;'
```

Finally you can run psql with:

```
psql -U wikijs -p 5432 -d wikijsdb -f /tmp/wikijs.sql
```