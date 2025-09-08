# 📦 Provisioning GCP with Crossplane on Windows (Minikube + Helm)

## 1. Prerequisites

* Windows 10/11 + PowerShell
* Minikube installed and running
* kubectl installed
* Docker Desktop
* gcloud CLI installed and configured
* Helm installed

## 2. Project File Structure

```
crossplane-minikube-gcp/
├─ provider-gcp.yaml           # Crossplane GCP Provider
├─ providerconfig-gcp.yaml     # GCP ProviderConfig (references credentials)
├─ resource-bucket.yaml        # GCP Bucket resource
├─ README.md                   # This file
├─ .gitignore                  # Ignore gcp-creds.json
└─ gcp-creds.json              # Service Account JSON (DO NOT version)
```

* `provider-gcp.yaml` → installs the GCP provider via Crossplane
* `providerconfig-gcp.yaml` → configures GCP project and credentials
* `resource-bucket.yaml` → defines the bucket to create
* `gcp-creds.json` → service account key (never commit this file)

## 3️. Ignore the JSON file in Git

Create a `.gitignore`:

```gitignore
# Ignore GCP credentials
gcp-creds.json
```

## 4️. Install Crossplane via Helm

```powershell
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

kubectl create namespace crossplane-system

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane

# Verify pods
kubectl get pods -n crossplane-system
```

## 5️. Create a GCP Service Account

* In GCP Console → IAM & Admin → Service Accounts → Create Service Account
* Assign Storage Admin role
* Download the JSON key: `gcp-creds.json`
* Check `projectID` in the JSON
* **Do not version this file**

## 6️. Create Kubernetes Secret

```powershell
kubectl create secret generic gcp-credentials `
  --namespace crossplane-system `
  --from-file=creds.json=C:\path\to\gcp-creds.json
```

## 7️. Create the GCP ProviderConfig

Set environment variable in PowerShell:

```powershell
$env:GCP_PROJECT="my-gcp-project"
```

Apply the ProviderConfig:

```powershell
kubectl apply -f providerconfig-gcp.yaml -n crossplane-system
```

## 8️. Install the GCP Provider

```powershell
kubectl apply -f provider-gcp.yaml -n crossplane-system
kubectl get providers.pkg.crossplane.io -n crossplane-system
```

## 9️. Create a GCP Bucket

```powershell
kubectl apply -f resource-bucket.yaml
kubectl get buckets
```

## 10. Verify the bucket in GCP

```powershell
gcloud auth activate-service-account --key-file="C:\path\to\gcp-creds.json"
gcloud storage buckets list
```

## 1️1. Cleanup

```powershell
kubectl delete bucket crossplanebucket
kubectl delete provider.pkg.crossplane.io provider-gcp-storage -n crossplane-system
kubectl delete providerconfig default -n crossplane-system
kubectl delete secret gcp-credentials -n crossplane-system
helm uninstall crossplane -n crossplane-system
kubectl delete namespace crossplane-system
```
