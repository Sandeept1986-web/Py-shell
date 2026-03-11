# GKE → Google Sheet Automation (Shell Script)

This script collects Kubernetes workload information from **GKE clusters in GCP** and writes the data into a **Google Sheet**.

The script fills columns **A–F** in the sheet with:

| Column | Field |
|------|------|
| A | Project ID |
| B | Cluster Name |
| C | Namespace |
| D | Type |
| E | K8s Object |
| F | Pods Running |

---

# Setup

## Make the script executable

```bash
chmod +x gke_to_sheet.sh
````

---

# Configuration

Before running the script, update the following variables inside the script.

---

## 1. GCP Projects

Specify the GCP projects that contain the GKE clusters.

```bash
PROJECT_IDS=(
  "CHANGE_ME_PROJECT_ID_1"
)
```

Replace with actual project IDs.

Example:

```bash
PROJECT_IDS=(
  "my-prod-project"
  "my-staging-project"
)
```

---

## 2. Spreadsheet ID

Set the **Google Spreadsheet ID**.

```bash
SPREADSHEET_ID="CHANGE_ME_SPREADSHEET_ID"
```

You can find it from the Google Sheet URL:

```
https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit#gid=...
```

Use only the part between:

```
/d/ and /edit
```

---

## 3. Sheet Tab Name

Specify the tab name inside the Google Sheet.

```bash
SHEET_NAME="CHANGE_ME_TAB_NAME"
```

Example:

```bash
SHEET_NAME="Monitoring Sheet"
```

---

## 4. Service Account JSON Path

Provide the path to the **service account JSON key** used to access Google Sheets.

```bash
SERVICE_ACCOUNT_FILE="CHANGE_ME_SERVICE_ACCOUNT_JSON_PATH"
```

Example:

```bash
SERVICE_ACCOUNT_FILE="/home/user/keys/sheet-writer-sa.json"
```

---

## 5. Included Namespaces

If you want only specific namespaces:

```bash
INCLUDED_NAMESPACES=(
  "default"
  "prod"
  "backend"
)
```

If you want to scan **all namespaces except excluded ones**, leave it empty:

```bash
INCLUDED_NAMESPACES=()
```

---

## 6. Excluded Namespaces

These namespaces are skipped by default:

```bash
EXCLUDED_NAMESPACES=(
  "kube-system"
  "kube-public"
  "kube-node-lease"
  "istio-system"
  "gke-gmp-system"
  "config-management-system"
)
```

Modify this list if needed.

---

## 7. Workload Types

If you want only deployments:

```bash
WORKLOAD_TYPES=(
  "deployments"
)
```

If you want deployments and statefulsets:

```bash
WORKLOAD_TYPES=(
  "deployments"
  "statefulsets"
)
```

Default example:

```bash
WORKLOAD_TYPES=(
  "deployments"
  "statefulsets"
  "daemonsets"
)
```

---

## 8. Clear Existing Data

If set to `true`, the script clears `A2:F` before writing new data.

```bash
CLEAR_EXISTING_DATA="true"
```

If you do not want old data cleared:

```bash
CLEAR_EXISTING_DATA="false"
```

---

# Required Tools

The following commands must be available where the script runs:

* `gcloud`
* `kubectl`
* `jq`
* `curl`
* `python3`

Verify installation:

```bash
gcloud version
kubectl version --client
jq --version
curl --version
python3 --version
```

---

# Required Access

## A. Access to GKE Data

The identity running the script must have permissions to:

* list GKE clusters
* get cluster credentials
* access Kubernetes API
* list namespaces
* list workloads
* list pods

Minimum commands that must work:

```bash
gcloud container clusters list
gcloud container clusters get-credentials
kubectl get ns
kubectl get deployments
kubectl get statefulsets
kubectl get daemonsets
kubectl get pods
```

If Kubernetes RBAC is restricted, cluster-level permissions must also be granted.

---

## B. Access to Google Sheets

The script writes to the sheet using a **Google Service Account**.

### Steps

1. Create a service account
2. Enable **Google Sheets API**
3. Download the JSON key
4. Share the target Google Sheet with the service account email as **Editor**

Example service account email:

```
sheet-writer@my-project.iam.gserviceaccount.com
```

Share the sheet with that email.

---

# Authentication Setup

Set the service account JSON path in the script:

```bash
SERVICE_ACCOUNT_FILE="/path/to/sheet-writer-sa.json"
```

The script uses:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="${SERVICE_ACCOUNT_FILE}"
gcloud auth application-default print-access-token
```

Ensure the JSON file is valid and readable.

---

# Running the Script

Run manually:

```bash
./gke_to_sheet.sh
```

Or:

```bash
bash gke_to_sheet.sh
```

---

# Recommended Places to Run

## Option 1: Cloud Shell

Fastest for testing.

Steps:

1. Open **GCP Cloud Shell**
2. Upload the script
3. Upload the service account JSON
4. Run:

```bash
chmod +x gke_to_sheet.sh
./gke_to_sheet.sh
```

---

## Option 2: Compute Engine VM

Recommended if the script must run on a schedule.

---

# Scheduling with Cron

Edit crontab:

```bash
crontab -e
```

Example: run every **30 minutes**

```cron
*/30 * * * * /bin/bash /path/to/gke_to_sheet.sh >> /path/to/gke_to_sheet.log 2>&1
```

This will:

* run the script every 30 minutes
* append logs to a file

