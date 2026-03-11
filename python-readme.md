# GKE → Google Sheet Automation

This script collects Kubernetes workload information from **GKE clusters in GCP** and automatically writes the data into a **Google Sheet**.

The script fills the sheet columns **A–F** with the following values:

| Column | Field                                   |
| ------ | --------------------------------------- |
| A      | Project ID                              |
| B      | Cluster Name                            |
| C      | Namespace                               |
| D      | Type (deployment/statefulset/daemonset) |
| E      | K8s Object                              |
| F      | Pods Running                            |

---

# Configuration

Before running the script, update the following variables inside the script.

---

## 1. GCP Projects

Specify the GCP projects that contain the GKE clusters.

```python
PROJECT_IDS = [
    "CHANGE_ME_PROJECT_ID_1",
]
```

Example:

```python
PROJECT_IDS = [
    "my-prod-project",
    "my-staging-project",
]
```

---

## 2. Spreadsheet ID

Set the **Google Spreadsheet ID**.

```python
SPREADSHEET_ID = "CHANGE_ME_SPREADSHEET_ID"
```

You can find it in the Google Sheet URL:

```
https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit#gid=...
```

Only use the portion between:

```
/d/ and /edit
```

---

## 3. Sheet Tab Name

Specify the tab name inside the Google Sheet.

```python
SHEET_NAME = "CHANGE_ME_TAB_NAME"
```

Example:

```python
SHEET_NAME = "Monitoring Sheet"
```

---

## 4. Service Account JSON Path

Provide the path to the **service account JSON key** used to access Google Sheets.

```python
SERVICE_ACCOUNT_FILE = "CHANGE_ME_SERVICE_ACCOUNT_JSON_PATH"
```

Example:

```python
SERVICE_ACCOUNT_FILE = "/home/user/keys/sheet-writer-sa.json"
```

---

## 5. Namespace Filters

If you want to scan **only specific namespaces**:

```python
INCLUDED_NAMESPACES = ["default", "prod", "backend"]
```

If you want the script to scan **all application namespaces**, leave it empty:

```python
INCLUDED_NAMESPACES = []
```

---

## 6. Workload Types

If you want only deployments:

```python
WORKLOAD_TYPES = ["deployments"]
```

If you want deployments and statefulsets:

```python
WORKLOAD_TYPES = ["deployments", "statefulsets"]
```

Default example:

```python
WORKLOAD_TYPES = ["deployments", "statefulsets", "daemonsets"]
```

---

# Required Access

## A. Access to GKE Data

The identity running the script must have permissions to:

* List GKE clusters
* Get cluster credentials
* Access Kubernetes API
* List namespaces
* List workloads
* List pods

Minimum commands that must work:

```
gcloud container clusters list
gcloud container clusters get-credentials
kubectl get ns
kubectl get deployments
kubectl get statefulsets
kubectl get daemonsets
kubectl get pods
```

If **Kubernetes RBAC** is restricted, cluster-level access must also be granted.

---

## B. Access to Google Sheets

A **Google Service Account** is required to update the sheet.

### Steps

1. Create a **Service Account**
2. Enable **Google Sheets API**
3. Download the **JSON key**
4. Share the target Google Sheet with the service account email as **Editor**

Example service account email:

```
sheet-writer@my-project.iam.gserviceaccount.com
```

Share the sheet with this email.

---

# Installation

Install required Python packages:

```
pip install google-api-python-client google-auth
```

---

# Running the Script

Run the script manually:

```
python3 gke_to_sheet.py
```

---

# Running on GCP

## Option 1: Cloud Shell

Fastest method for testing.

Steps:

1. Open Cloud Shell
2. Upload script
3. Run:

```
python3 gke_to_sheet.py
```

---

## Option 2: Compute Engine VM

Recommended for **scheduled automation**.

### Create a small VM

Example:

```
e2-micro
```

Install Python dependencies and place the script on the VM.

---

# Scheduling with Cron

Edit crontab:

```
crontab -e
```

Example: run every **30 minutes**

```
*/30 * * * * /usr/bin/python3 /path/to/gke_to_sheet.py >> /path/to/gke_to_sheet.log 2>&1
```

This will:

* Run the script every 30 minutes
* Log output to a file

---

# Summary

This automation will:

1. Discover **GKE clusters in specified projects**
2. Connect to each cluster
3. Scan namespaces and workloads
4. Count running pods
5. Write results into the **Google Sheet**
