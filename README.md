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

