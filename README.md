# 📦 Provisioning GCP with Crossplane on Windows (Minikube + Helm)

## Prerequisites

- Windows 10/11 + PowerShell  
- Minikube installed and running  
- kubectl installed  
- Docker Desktop  
- gcloud CLI installed and configured  
- Helm installed  

---

## Project File Structure

crossplane-minikube-gcp/
├─ provider-gcp.yaml # Crossplane GCP Provider
├─ providerconfig-gcp.yaml # GCP ProviderConfig (references credentials)
├─ resource-bucket.yaml # GCP Bucket resource
├─ README.md # This file
├─ .gitignore # Ignore gcp-creds.json
└─ gcp-creds.json # Service Account JSON (DO NOT version)


- `provider-gcp.yaml` → installs the GCP provider via Crossplane  
- `providerconfig-gcp.yaml` → configures GCP project and credentials  
- `resource-bucket.yaml` → defines the bucket to create  
- `gcp-creds.json` → service account key (never commit this file)  

---

## Ignore the JSON file in Git

Create a `.gitignore`:

```gitignore
# Ignore GCP credentials
gcp-creds.json

---

## Install Crossplane via Helm

```powershell
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

kubectl create namespace crossplane-system

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane

# Verify pods
kubectl get pods -n crossplane-system

## Create a GCP Service Account

1. In GCP Console → IAM & Admin → Service Accounts → Create Service Account  
2. Assign **Storage Admin** role  
3. Download the JSON key: `gcp-creds.json`  
4. Check `projectID` in the JSON  
5. **Do not version this file**  

---

## Create Kubernetes Secret

```powershell
kubectl create secret generic gcp-credentials `
  --namespace crossplane-system `
  --from-file=creds.json=C:\path\to\gcp-creds.json

---

## Create the GCP ProviderConfig

1. Set the GCP project environment variable in PowerShell:

```powershell
$env:GCP_PROJECT="my-gcp-project"

2. Apply the ProviderConfig:

```powershell
kubectl apply -f providerconfig-gcp.yaml -n crossplane-system

---

## Install the GCP Provider

1. Apply the provider YAML:

```powershell
kubectl apply -f provider-gcp.yaml -n crossplane-system

2. Verify the provider installation:

```powershell
kubectl get providers.pkg.crossplane.io -n crossplane-system

## Create a GCP Bucket

1. Apply the bucket resource YAML:

```powershell
kubectl apply -f resource-bucket.yaml -n crossplane-system
Verify that the bucket has been created:

```powershell
kubectl get buckets -n crossplane-system

---

## Verify the Bucket in GCP

1. Authenticate with your service account:

```powershell
gcloud auth activate-service-account --key-file="C:\path\to\gcp-creds.json"
List your GCP buckets to confirm creation:

```powershell
gcloud storage buckets list

---

## Cleanup

```powershell
kubectl delete bucket crossplanebucket -n crossplane-system
kubectl delete provider.pkg.crossplane.io provider-gcp-storage -n crossplane-system
kubectl delete providerconfig default -n crossplane-system
kubectl delete secret gcp-credentials -n crossplane-system
helm uninstall crossplane -n crossplane-system
kubectl delete namespace crossplane-system
