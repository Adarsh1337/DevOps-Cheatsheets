# GCP Cheatsheet

## Core Services Overview
- Compute Engine (GCE): VMs
- Cloud Storage (GCS): Object storage
- Google Kubernetes Engine (GKE): Managed Kubernetes
- Cloud IAM: Identity and permissions
- Cloud SQL/Spanner/BigQuery: Managed databases and analytics

## gcloud CLI
```bash
# Authenticate and set project
gcloud auth login
gcloud config set project PROJECT_ID

# Set region/zone
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
```

## Compute Engine (GCE)
```bash
# List machine types
gcloud compute machine-types list --zones=us-central1-a

# Create VM
gcloud compute instances create vm1 \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=http-server

# Open firewall rule
gcloud compute firewall-rules create allow-http --allow tcp:80 --target-tags http-server --direction INGRESS

# SSH
gcloud compute ssh vm1 --zone us-central1-a
```

## Cloud Storage (GCS)
```bash
# Create bucket
gsutil mb -l US-CENTRAL1 gs://my-bucket-$(date +%s)/

# Upload/Sync
gsutil cp file.txt gs://my-bucket/
gsutil -m rsync -r ./site gs://my-bucket/site

# Make object public
gsutil acl ch -u AllUsers:R gs://my-bucket/file.txt
```

## GKE
```bash
# Create cluster
gcloud container clusters create-auto app --region us-central1

# Get credentials
gcloud container clusters get-credentials app --region us-central1

# Validate
kubectl get nodes -o wide
kubectl get pods -A
```

## IAM
```bash
# List service accounts
gcloud iam service-accounts list

# Create service account and key
gcloud iam service-accounts create ci-sa --display-name "CI Service Account"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member serviceAccount:ci-sa@$PROJECT_ID.iam.gserviceaccount.com \
  --role roles/viewer
```

## Serverless
- Cloud Functions, Cloud Run, App Engine

## Networking
- VPC, subnets, firewall rules, routes, load balancers, Cloud NAT

## Best Practices
- Use least privilege IAM
- Labels for resource organization
- Use Secret Manager
- Use IaC (Deployment Manager/Terraform)
- Enable Cloud Audit Logs and Monitoring
