# azure-billing-cost-optimization
Solution for Azure Serverless Cost Optimization Assignment – Symplique Solutions

# 💰 Azure Serverless Cost Optimization – Billing Records Archival

## 🔧 Problem Statement

The Cosmos DB used for billing records has grown to over 2 million records, increasing storage and throughput costs. We need a cost-effective and seamless way to reduce expenses by archiving older records while keeping API contracts unchanged.

---

## 🎯 Solution Summary

We split data into two tiers:

- **Hot Tier**: Azure Cosmos DB – last 3 months of records (frequently accessed)
- **Cold Tier**: Azure Blob Storage – archive for older records

### 💡 Key Benefits

- Reduces Cosmos DB costs (storage + RU/s)
- No data loss, no downtime
- API remains unchanged (via cold read fallback)

---

## 🏗️ Architecture Diagram

![Solution Architecture](architecture/solution_diagram.png)

---

## 🛠️ Components

| Component | Purpose |
|----------|---------|
| **Cosmos DB** | Stores recent billing records |
| **Azure Blob Storage** | Stores archived records in JSON format |
| **Azure Function (Timer)** | Archives records older than 90 days |
| **Azure Function (Cold Read)** | Fetches archived records if not found in Cosmos DB |

---

## 🗃️ Folder Structure

azure-billing-cost-optimization/
├── architecture/
├── scripts/
│ ├── archive_old_billing_records.py
│ └── cold_read_fallback.py
├── pseudocode/
├── README.md
├── prompts_chatgpt.md
└── LICENSE



---

## 🚀 How It Works

### 🔁 Archiving (Timer Trigger)

- Scans Cosmos DB for records older than 90 days
- Moves them to Blob Storage (as compressed JSON)
- Deletes from Cosmos DB to reduce cost

### 🔍 Cold Read Logic

- When an old record is requested:
  - Function checks Cosmos DB
  - If not found, fetches from Blob Storage and returns

---

## 📜 Usage

```bash
# Archive script
python3 scripts/archive_old_billing_records.py

# Cold read fallback test
python3 scripts/cold_read_fallback.py

azure-billing-cost-optimization/
├── architecture/
│   └── solution_diagram.png            # Diagram of the Hot-Cold tiering
│
├── scripts/
│   ├── archive_old_billing_records.py  # Moves old records to Cool/Archive
│   └── cold_read_fallback.py           # Reads from cold if hot not available
│
├── pseudocode/
│   ├── data_archival.md                # Step-by-step logic in plain text
│   └── cold_read_strategy.md
│
├── bicep/
│   └── main.bicep                      # Infra-as-Code for Blob + CosmosDB
│
├── pipelines/
│   └── azure-pipeline.yml              # Optional Azure DevOps Pipeline
│
├── prompts_chatgpt.md                 # ChatGPT planning & guidance logs
├── requirements.txt                   # If Python scripts use packages
├── README.md
└── LICENSE                            # MIT or Apache-2.0


README.md (Sample)

# 💰 Azure Billing Cost Optimization with Serverless and Tiered Storage

This project implements a **hot-cold data tiering architecture** using Azure Blob Storage and Cosmos DB, designed to reduce storage costs and support on-demand data archival for billing records.

## 🔧 Features

- Serverless approach (Functions/Logic Apps ready)
- Blob Storage tiering: Hot → Cool → Archive
- Cosmos DB for fast transactional reads
- Python scripts to automate archival & cold-read fallback
- Infrastructure-as-Code with Bicep
- Optional Azure Pipeline for CI/CD
- Free Tier Friendly

## 📁 Project Structure

| Folder        | Purpose                            |
|---------------|-------------------------------------|
| `architecture/` | Visual diagram of architecture     |
| `scripts/`     | Archival and fallback scripts       |
| `pseudocode/`  | High-level logic and explanations   |
| `bicep/`       | Infrastructure definitions          |
| `pipelines/`   | Azure DevOps YAML for automation    |
| `prompts_chatgpt.md` | Prompt-engineering log         |

## 🖼 Architecture Diagram

![Solution Diagram](architecture/solution_diagram.png)

## 🚀 How It Works

1. **Archive Script**: Moves old billing records (>90 days) to Cool or Archive tiers.
2. **Fallback Logic**: If data is missing in Hot, script auto-fetches from Cold storage.
3. **Cosmos DB**: Optional for real-time reads + NoSQL performance.
4. **Bicep Deployment**: Easily provision storage accounts and DBs.
5. **CI/CD**: Optional pipeline to run/test/archive on push.

## 🧪 Test Locally

No Azure account needed — you can mock data or use [Azurite](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite) for local blob emulation.

## ✅ To-Do Checklist

- [x] Design architecture
- [x] Implement archival script
- [x] Implement cold-read fallback
- [x] Write pseudocode docs
- [x] Create Bicep templates
- [ ] Add CI/CD pipeline
- [ ] Deploy to Azure

## 🆓 Free Tier Notes

- Cosmos DB free tier: 400 RU/s, 5GB
- Blob Storage: 5GB in general-purpose v2 (Hot/Cool/Archive supported)
- No VMs or paid services used

## 📄 License

MIT License

⚙️ Azure DevOps Pipeline
pipelines/azure-pipeline.yml

trigger:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.x'

  - script: |
      pip install -r requirements.txt
      python scripts/archive_old_billing_records.py
    displayName: 'Run Archive Script'

  - script: |
      python scripts/cold_read_fallback.py
    displayName: 'Run Cold Read Fallback Script'
