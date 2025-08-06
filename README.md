Azure Cosmos DB Cost Optimization with Serverless Architecture

Introduction:

The challenge is to optimize costs for a growing Azure Cosmos DB database in a serverless architecture. The service stores billing records, is read-heavy, and older records (3 months) are rarely accessed but must remain available with a response time in the order of seconds. Each record is up to 300 KB, and the database has over 2 million records. The solution must be simple, seamless, and maintain existing API contracts.

Proposed Solution: Tiered Data Storage with Azure Blob Storage

The core of the solution is to implement a tiered storage strategy. We will leverage Azure Blob Storage, a cost-effective solution for storing large amounts of unstructured data, for archiving older, less frequently accessed records. This approach significantly reduces the Cosmos DB Request Units (RUs) and storage costs while maintaining data availability.

High-Level Architecture:

Hot Tier (Cosmos DB): Active records (less than 3 months old) will reside in Azure Cosmos DB. This provides low-latency access for real-time operations.

Cold Tier (Blob Storage): Records older than three months will be moved from Cosmos DB to an Azure Blob Storage container. Blob Storage offers different access tiers (Hot, Cool, Archive) that can be utilized for further cost optimization. The Cool or Archive tier would be ideal for this use case.

Data Archival Process: An automated process, likely an Azure Function or a Logic App, will periodically scan the Cosmos DB for records older than three months and move them to Blob Storage.

Data Retrieval Process: The existing API will be enhanced to handle requests for both hot and cold data transparently.

If a record is found in Cosmos DB, it is returned directly.

If not found, the API will query Blob Storage for the record. This introduces a slight latency but is acceptable given the requirement for "seconds" of response time.

Detailed Architecture:

Frontend/API: A single Azure Function acts as the unified API for reading and writing billing records. It accepts a recordId.

Azure Cosmos DB: The primary data store for recent records.

Azure Blob Storage: The archival data store. The blob storage container can be structured with a logical path like year/month/recordId.json to facilitate easy retrieval and management.

Archival Azure Function: A timer-triggered Azure Function that runs daily. It will query Cosmos DB for records with a timestamp older than three months. For each record, it will:

Read the record from Cosmos DB.

Serialize the record to a JSON file.

Upload the JSON file to Blob Storage.

Delete the record from Cosmos DB.

Data Retrieval Logic: The read API function will first try to retrieve the record from Cosmos DB. If the query returns no result, it will then attempt to retrieve the corresponding file from Blob Storage.

Implementation Details:

API Contract: No changes are required. The API GET /billing/{recordId} will function as before. The underlying implementation will handle the tiered storage logic.

Archival Function (Pseudocode):

Code snippet

func ArchiveOldRecords(timerTrigger: TimerInfo):
  cutoff_date = datetime.now() - timedelta(days=90)
  cosmos_db_query = "SELECT * FROM c WHERE c.timestamp < @cutoffDate"
  records_to_archive = cosmos_db_client.query_items(cosmos_db_query, parameters=[{'name': '@cutoffDate', 'value': cutoff_date.isoformat()}])

  for record in records_to_archive:
    try:
      blob_name = f"{record.timestamp.year}/{record.timestamp.month}/{record.id}.json"
      blob_client.upload_blob(blob_name, json.dumps(record))
      cosmos_db_client.delete_item(record.id)
    except Exception as e:
      # Log error and handle retries
      log_error(f"Failed to archive record {record.id}: {e}")
Read API Function (Pseudocode):

Code snippet

func GetRecord(request: HttpRequest, recordId: str):
  // Try to read from Cosmos DB
  try:
    record = cosmos_db_client.read_item(recordId)
    if record:
      return JsonResponse(record)
  except ItemNotFoundError:
    // Not in Cosmos DB, try Blob Storage
    try:
      blob_name = get_blob_name_from_recordId(recordId)
      blob_data = blob_client.download_blob(blob_name).readall()
      record = json.loads(blob_data)
      return JsonResponse(record)
    except BlobNotFoundError:
      return HttpResponse(status_code=404)
    except Exception as e:
      log_error(f"Error retrieving record {recordId} from Blob: {e}")
      return HttpResponse(status_code=500)
Potential Problems & Mitigations:

Data Inconsistency: A race condition could occur if a record is requested during the archival process. The read API might fail to find it in Cosmos DB but the Blob hasn't been created yet.

Mitigation: The archival process should first upload the blob and only then delete the item from Cosmos DB. A robust retry mechanism is essential.

Archival Function Performance: Processing 2 million records could take a long time and incur significant costs.

Mitigation: The archival function should be designed to run in batches. It can use continuation tokens for querying Cosmos DB and process records in chunks to avoid timeouts and excessive memory usage.

Latency for Old Records: While a response time in the order of "seconds" is acceptable, cold starts for the Azure Function and network latency to Blob Storage could impact performance.

Mitigation: We can utilize a small, in-memory cache in the Azure Function for frequently accessed old records to further optimize retrieval. We can also use a "warm" Blob storage tier.

Security: Billing records are sensitive.

Mitigation: Both Cosmos DB and Blob Storage must be secured with proper access controls (e.g., Managed Identities, Role-Based Access Control). Data in Blob Storage should be encrypted at rest (which is a default feature) and transit.

                        +---------------------------+
                        |     Azure Function App    |
                        |  (Billing API Layer)      |
                        +------------+--------------+
                                     |
            +------------------------+------------------------+
            |                                                 |
     +------v--------+                                 +------v--------+
     | Cosmos DB     | (BillingRecords-Hot)           | Azure Blob     |
     | Hot Container |  (Latest 3 Months)             | Archive        |
     +---------------+                                +---------------+
            |                                                 |
     +------v--------+                                +------v--------+
     | Azure Function | (Cold Fetch Proxy)           | Durable Archive|
     +---------------+                                +---------------+

                 +---------> App Logic (No API Changes)

🔄 Data Archival Logic (Pseudocode)
python
Copy
Edit

def archive_old_billing_records():
    # Connect to Cosmos DB
    for record in cosmos_db.query("SELECT * FROM c WHERE c.timestamp < now - 90d"):
        save_to_blob(record)
        delete_from_cosmos(record)
        
🔁 Cold Read Function (Pseudocode)

def get_billing_record(record_id):
    record = cosmos_db.read(record_id)
    if not record:
        record = blob_storage.search_by_id(record_id)
    return record

    
💰 Cost Optimization Strategy

| Resource     | Tier                         | Reason                      |
| ------------ | ---------------------------- | --------------------------- |
| Cosmos DB    | Serverless                   | Only pay per request (RU/s) |
| Blob Storage | Cool/Archive                 | 80%+ cheaper than Cosmos DB |
| Functions    | Consumption                  | Scales to 0, minimal cost   |
| Indexing     | Turned off for archived data | Saves cost                  |  

💥 Failure Points & Handling
| Problem                        | Solution                                                              |
| ------------------------------ | --------------------------------------------------------------------- |
| Function failure during move   | Use Durable Functions with retry + logging                            |
| Data inconsistency             | Mark archived records with metadata (`isArchived=true`) before delete |
| Latency too high in Blob fetch | Cache recent cold fetches temporarily using Redis or local cache      |
| High retrieval from blob       | Add CDN or small Cosmos Archive cache if same record is fetched often |


<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/94c66218-8cf7-47ae-9228-67773496973d" />

