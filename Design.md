### ✅ Final Kafka-Based Data Creation Flow (Step-by-Step, Word-by-Word)

---

#### **1. Upload Sheet**
You upload a sheet with data rows.  
You also give a `referenceId` (like `cmp-123`) and a `processName` (like `facilityCreation`).

---

#### **2. Identify Unique Rows**
For each row in the sheet:  
Extract a unique value (like mobile number) as `uniqueIdentifier`.  
Check if this identifier with the same `referenceId` already exists in the system.

---

#### **3. Insert New Rows**
If it does not exist:  
Save the row in a tracking table with:
- status = `pending`
- type = `processName` (like facilityCreation)
- referenceId = your input (like cmp-123)
- rowData = original sheet row in JSON

---

#### **4. Retry Old Failed Rows**
Check for rows with the same `referenceId` where status = `failed`.  
Change their status to `pending` so they can be retried.

---

#### **5. Send Pending Rows to Kafka**
Take 100 rows at a time with status = `pending`.  
For each batch:
- Send all 100 rows to Kafka.
- Along with each row, send:
  - referenceId
  - processName
  - currentBatchNumber (like 2)
  - totalBatchNumber (like 10)

---

#### **6. Kafka Consumer Processing**
Kafka consumer receives each row.  
Based on `processName`, it creates the required data (like a facility).  
If creation is successful, update the status to `success`.

---

#### **7. Wait and Validate**
After all batches are sent, wait for a short time (like 1 minute).  
This is to allow delayed Kafka processing to finish.

---

#### **8. Final Check**
Check if any row with that `referenceId` still has status = `pending` or `failed`.  
If yes, mark the overall process as `failed`.  
If all rows are `success`, mark it as `completed`.

---

#### **9. Retry if Needed**
You can retry later.  
Failed rows will be picked up again as `pending`.  
No duplicates will be created because of the uniqueIdentifier check.

---
