🚀 Azure Cost Optimization: Serverless Hot-Cold Billing Data Tiering
✅ Problem Statement
We have a serverless architecture using Azure Cosmos DB to store billing records (up to 300 KB each, over 2 million total). The system is read-heavy, but records older than 3 months are rarely accessed.

Challenge: Cosmos DB costs are increasing due to data growth.
Goal: Reduce costs without affecting availability, data integrity, or existing APIs.

✅ Key Requirements
Requirement	Fulfilled ✅
No data loss / No downtime	✅
No changes to existing API	✅
Hot-Cold archival strategy	✅
Fast retrieval of cold data	✅
Simplicity & easy maintenance	✅

💡 Solution Overview
We propose a Hot-Cold Tiered Storage Architecture:

Hot Data (last 90 days) → Azure Cosmos DB

Cold Data (older than 90 days) → Azure Blob Storage (Cold/Archive Tier)

A Python-based Archiver moves data to Blob daily.

API remains unchanged with a Fallback Read Logic:

First query Cosmos DB.

If not found, fallback to Blob Storage.

🧱 Architecture Diagram
sql
Copy
Edit
Client/API
   |
   |-- Read / Write Request
   |
Azure Function App (API Layer)
   |
   |--- Cosmos DB (Hot data)
   |
   |--- If not found → Blob Storage (Cold data)
          |
          |--- Cold read via fallback
Scheduled Data Mover: Python script (daily via Azure Function or GitHub Action)

🧑‍💻 Core Scripts
1. Archival Script: archive_old_records.py
Moves records older than 90 days from Cosmos DB → Blob.

python
Copy
Edit
from datetime import datetime, timedelta
from azure.cosmos import CosmosClient
from azure.storage.blob import BlobServiceClient
import json, os

# Environment configs
COSMOS_URL = os.getenv("COSMOS_URL")
COSMOS_KEY = os.getenv("COSMOS_KEY")
DATABASE_NAME = "BillingDB"
CONTAINER_NAME = "Records"

BLOB_CONN_STRING = os.getenv("AZURE_STORAGE_CONNECTION_STRING")
BLOB_CONTAINER = "cold-billing-records"

# Cutoff: 90 days
cutoff_date = datetime.utcnow() - timedelta(days=90)

# Cosmos client
cosmos_client = CosmosClient(COSMOS_URL, credential=COSMOS_KEY)
container = cosmos_client.get_database_client(DATABASE_NAME).get_container_client(CONTAINER_NAME)

# Blob client
blob_client = BlobServiceClient.from_connection_string(BLOB_CONN_STRING)
blob_container = blob_client.get_container_client(BLOB_CONTAINER)

# Move old records
for record in container.query_items(f"SELECT * FROM c WHERE c.timestamp < '{cutoff_date.isoformat()}'", enable_cross_partition_query=True):
    blob_name = f"{record['id']}.json"
    blob_container.upload_blob(blob_name, json.dumps(record), overwrite=True)
    container.delete_item(item=record['id'], partition_key=record['partitionKey'])
2. Fallback Read Logic (API Layer)
In your existing GET /billing/{id} function:

python
Copy
Edit
def get_billing_record(record_id):
    try:
        return cosmos_container.read_item(item=record_id, partition_key=record_id)
    except Exception:
        # Fallback to Blob
        blob_name = f"{record_id}.json"
        blob_data = blob_container_client.get_blob_client(blob=blob_name).download_blob().readall()
        return json.loads(blob_data)
⚙️ Optional GitHub Actions (Free Tier)
Automate archival with a GitHub Action:

yaml
Copy
Edit
# .github/workflows/archive.yaml
name: Archive Old Billing Records

on:
  schedule:
    - cron: '0 0 * * *'  # Every day at midnight UTC

jobs:
  archive:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install azure-cosmos azure-storage-blob
      - name: Run archival script
        env:
          COSMOS_URL: ${{ secrets.COSMOS_URL }}
          COSMOS_KEY: ${{ secrets.COSMOS_KEY }}
          AZURE_STORAGE_CONNECTION_STRING: ${{ secrets.BLOB_CONN_STRING }}
        run: python archive_old_records.py
🔒 Security & Cost Considerations
Cold tier Blob Storage is ~80% cheaper than Cosmos DB.

Azure Function or GitHub Action handles archiving without server overhead.

Secrets managed via GitHub Secrets or Azure Key Vault.

📁 Repo Structure
bash
Copy
Edit
.
├── archive_old_records.py
├── prompts_chatgpt.md      # This ChatGPT conversation
├── fallback_api_sample.py
├── .github
│   └── workflows
│       └── archive.yaml
├── architecture.png         # (Optional image)
└── README.md

✅ Azure Cost Optimization Summary

| 📌 **Category** | 📝 **Details**                                                                        |
| --------------- | ------------------------------------------------------------------------------------- |
| **Challenge**   | Cosmos DB costs rising due to 2M+ records, most >90 days old. APIs must not change.   |
| **Solution**    | Move old records to Blob Storage via automated functions. Keep hot data in Cosmos DB. |
| **Benefit**     | Saves cost (storage + RU), seamless to user, no downtime, easy to scale + maintain.   |


![Tiered Architecture](https://github.com/user-attachments/assets/94c66218-8cf7-47ae-9228-67773496973d)
![Cold-Read Flowchart](https://github.com/user-attachments/assets/853925c3-68c4-4248-bfd6-8f01300eaa4a)
