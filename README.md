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

