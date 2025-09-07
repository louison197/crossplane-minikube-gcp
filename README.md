# 📦 Provisioning GCP with Crossplane on Windows (Minikube + Helm)

## 1️⃣ Prerequisites

* Windows 10/11 + PowerShell
* Minikube installed and running
* kubectl installed
* Docker Desktop
* gcloud CLI installed and configured
* Helm installed

## 2️⃣ Project File Structure

```
crossplane-minikube-gcp/
├─ provider-gcp.yaml           # Crossplane GCP Provider
├─ providerconfig-gcp.yaml     # GCP ProviderConfig (references credentials)
├─ resource-bucket.yaml        # GCP Bucket resource
├─ README.md                   # This file
├─ .gitignore                  # Ignore gcp-creds.json
└─ gcp-creds.json              # Service Account JSON (DO NOT version)
```

* **provider-gcp.yaml** → installs the GCP provider via Crossplane
* **providerconfig-gcp.yaml** → configures GCP project and credentials
* **resource-bucket.yaml** → defines the bucket to create
* **gcp-creds.json** → service account key (never commit this file)

## 3️⃣ Ignore the JSON file in Git

Create a `.gitignore` file:

```
# Ignore GCP credentials
gcp-creds.json
```

## 4️⃣ Install Crossplane via Helm

1. Add the Crossplane Helm repo and update:

```powershell
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
```

2. Create the Crossplane namespace:

```powershell
kubectl create namespace crossplane-system
```

3. Install Crossplane:

```powershell
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

4. Verify pods:

```powershell
kubectl get pods -n crossplane-system
```

## 5️⃣ Create a GCP Service Account

* Go to **GCP Console → IAM & Admin → Service Accounts → Create Service Account**
* Assign **Storage Admin** role
* Download the JSON key as `gcp-creds.json`
* Check the `projectID` in the JSON
* **Do not commit this file to Git**

## 6️⃣ Create Kubernetes Secret

```powershell
kubectl create secret generic gcp-credentials `
  --namespace crossplane-system `
  --from-file=creds.json=C:\path\to\gcp-creds.json
```

## 7️⃣ Create the GCP ProviderConfig

* Reference your project name via the environment variable:

```powershell
$env:GCP_PROJECT="my-gcp-project"
```

* Apply the `providerconfig-gcp.yaml`:

```powershell
kubectl apply -f providerconfig-gcp.yaml -n crossplane-system
```

*(The YAML already contains the projectID and secret reference.)*

## 8️⃣ Install the GCP Provider

```powershell
kubectl apply -f provider-gcp.yaml -n crossplane-system
kubectl get providers.pkg.crossplane.io -n crossplane-system
```

*(The YAML already specifies the provider package.)*

## 9️⃣ Create a GCP Bucket

```powershell
kubectl apply -f resource-bucket.yaml -n crossplane-system
kubectl get buckets -n crossplane-system
```

*(The YAML already defines the bucket name, location, and providerConfigRef.)*

## 🔟 Verify the Bucket in GCP

1. Authenticate with the service account:

```powershell
gcloud auth activate-service-account --key-file="C:\path\to\gcp-creds.json"
```

2. List buckets:

```powershell
gcloud storage buckets list
```

## 1️⃣1️⃣ Cleanup

```powershell
kubectl delete bucket crossplanebucket -n crossplane-system
kubectl delete provider.pkg.crossplane.io provider-gcp-storage -n crossplane-system
kubectl delete providerconfig default -n crossplane-system
kubectl delete secret gcp-credentials -n crossplane-system
helm uninstall crossplane -n crossplane-system
kubectl delete namespace crossplane-system
```

---

✅ **Notes**

* Never commit your `gcp-creds.json` file.
* Replace `my-gcp-project` with your actual GCP project name.
* All YAML files contain the necessary configuration for Crossplane to provision resources.

