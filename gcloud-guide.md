# Google Cloud SDK (`gcloud`) Guide

This page is a quick reference for common `gcloud` commands and workflows on Ubuntu.

## Setup and auth

```bash
gcloud version
gcloud info
gcloud init
```

Sign in (user):

```bash
gcloud auth login
gcloud auth list
```

Application Default Credentials (useful for local development with Google client libraries):

```bash
gcloud auth application-default login
gcloud auth application-default print-access-token
```

## Projects, config, region/zone

```bash
gcloud projects list
gcloud config set project <PROJECT_ID>
gcloud config get-value project

gcloud config set compute/region <REGION>
gcloud config set compute/zone <ZONE>
gcloud config list
```

Multiple configurations (e.g., work vs personal):

```bash
gcloud config configurations list
gcloud config configurations create <NAME>
gcloud config configurations activate <NAME>
```

## Useful “list” commands

```bash
gcloud services list --enabled
gcloud compute instances list
gcloud compute disks list
gcloud compute networks list
gcloud iam service-accounts list
```

## Enable APIs

```bash
gcloud services enable run.googleapis.com
gcloud services enable artifactregistry.googleapis.com
```

## Service accounts (common IAM workflow)

Create a service account:

```bash
gcloud iam service-accounts create <SA_NAME> \
  --display-name "<DISPLAY_NAME>"
```

Grant a role (example: Artifact Registry reader) to a principal:

```bash
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:<SA_NAME>@<PROJECT_ID>.iam.gserviceaccount.com" \
  --role="roles/artifactregistry.reader"
```

Create a key file (only if you must; prefer Workload Identity / ADC where possible):

```bash
gcloud iam service-accounts keys create key.json \
  --iam-account="<SA_NAME>@<PROJECT_ID>.iam.gserviceaccount.com"
```

## Compute Engine

SSH into a VM:

```bash
gcloud compute ssh <INSTANCE_NAME> --zone <ZONE>
```

Copy files:

```bash
gcloud compute scp local_file <INSTANCE_NAME>:~/ --zone <ZONE>
gcloud compute scp <INSTANCE_NAME>:~/remote_file . --zone <ZONE>
```

Start/stop:

```bash
gcloud compute instances start <INSTANCE_NAME> --zone <ZONE>
gcloud compute instances stop <INSTANCE_NAME> --zone <ZONE>
```

## Cloud Run (quick deploy)

```bash
gcloud run deploy <SERVICE_NAME> \
  --source . \
  --region <REGION> \
  --allow-unauthenticated
```

## Artifact Registry + Docker

Authenticate Docker to push/pull from Artifact Registry:

```bash
gcloud auth configure-docker <REGION>-docker.pkg.dev
```

## Output formatting and troubleshooting

```bash
gcloud help
gcloud <COMMAND> --help

gcloud compute instances list --format="table(name,zone,status)"
gcloud logging read 'resource.type="gce_instance"' --limit=20
```
