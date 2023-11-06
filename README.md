# ghostgke
Deploy Ghost Blog on GKS with Persistent Volume Claim and Cloud SQL

# deployment notes

1. Create the Project
2. Create the GKE Autopilot cluster (dependent on deployment requirements)
3. MySQL deployment (PVC, Deployment, Service), moved away from the Wordpress example and CloudSQL to deploying MySQL on GKE
4. MySQL database creation (execing into the contained, can look at a better way to provision)
5. Ghost deployment (PVC, Deployment, Service, Ingress)

# still to work on

1. TLS for the Ingress controller using LetsEncrypt
2. Database provisioning and backups (tiredofit)

# detailed steps

1. Create the cluster:  

```console
CLUSTER_NAME=<INSERT_CLUSTER_NAME>
gcloud container clusters create-auto $CLUSTER_NAME
```

2. Deploy MySQL & Prep the Database

create the Root password as a secret
```console
kubectl create secrete generic mysql --from-literal=rootpassword=<INSERT_ROOT_PASSWORD>
```

create the MySQL persistent volume claim
```console
kubectl apply -f mysql-volumeclaim.yaml
```

deploy the MySQL instance
```console
kubectl apply -f mysql.yaml
```

expose the MySQL instance through a service
```console
kubectl apply -f mysql-service.yaml
```

create the database and database user by exec'ing into the Pod and useing MySQL command line
```console
kubectl exec -t <MYSQL_POD_NAME> /bin/bash
CREATE USER '<DATABASE_USER>'@'%' WITH mysql_native_password IDENTIFIED BY '<DATABASE_PASSWORD>';
CREATE DATABASE <DATABASE_NAME>; # should be same as username to keep it simple
GRANT ALL PRIVILEGES ON <DATABASE_NAME>.* TO '<DATABASE_USER>'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

create the ghost secret
```console
kubectl create secret generic <mysql-GHOST_SITE_NAME> --from-literal=url=<GHOST_SITE_URL> --from-literal=database__connection__database=<DATABASE_NAME> --from-literal=database__connection__user=<DATABASE_USER> --from-literal=database__connection__password=<DATABASE_PASSWORD> --from-literal=database__connection__host=<MYSQL_K8S_SERVICE_NAME>
```

3. Deploy Ghost and confirm it's running (using logs, it's not exposed yet but save wasted troubleshooting effort!)

Create Ghost persistent volume claim
```console
kubectl apply -f ghost-volumeclaim.yaml
```

Deploy Ghost
```console
kubectl apply -f ghost.yaml
```


Prepare for Google Managed Certificates

Create the external IP (needed to configure DNS provider for Ghost URL's)
```console
gcloud compute addresses create <EXTERNAL_IP_NAME> --global
```
Retrieve the external IP and go configure DNS for the Ghost blog
```console
gcloud compute addresses describe ghost-external-ip --global
```

