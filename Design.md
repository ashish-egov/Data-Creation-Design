## ✅ Kafka-Based Data Creation Flow (Standardized)

---

### **1. Upload Data Sheet**
Upload a data sheet containing rows of information.  
Provide a `referenceId` (e.g., `cmp-123`) and a `processName` (e.g., `facilityCreation`).

---

### **2. Identify Unique Entries**
For each row in the sheet:  
Extract a unique identifier (e.g., mobile number) as `uniqueIdentifier`.  
Check if this identifier, along with the same `referenceId`, already exists in the system.

---

### **3. Insert New Records**
If the record does not exist:  
Insert the row into a tracking table with the following properties:
- `status = pending`
- `type = processName` (e.g., `facilityCreation`)
- `referenceId = input referenceId` (e.g., `cmp-123`)
- `rowData = original sheet row in JSON format`

---

### **4. Retry Failed Entries**
Check for rows with the same `referenceId` where the `status = failed`.  
Change their status to `pending` so they can be retried.

---

### **5. Publish Pending Rows to Kafka**
Select 100 rows at a time with `status = pending`.  
For each batch:
- Send all 100 rows to Kafka.
- Include the following metadata with each row:
  - `referenceId`
  - `processName`
  - `currentBatchNumber` (e.g., 2)
  - `totalBatchNumber` (e.g., 10)

---

### **6. Kafka Consumer Processing**
Kafka consumer processes each row.  
Based on the `processName`, the required data (e.g., facility) is created.  
If the creation is successful, update the `status` to `success`.

---

### **7. Wait and Validate**
After all batches are sent, wait for a short period (e.g., 1 minute).  
This provides time for delayed Kafka processing to complete.

---

### **8. Final Validation**
Check for any rows with the given `referenceId` that still have `status = pending` or `failed`.  
If such rows exist, mark the overall process as `failed`.  
If all rows are marked as `success`, mark the process as `completed`.

---

### **9. Retry Mechanism**
You can retry the process later.  
Failed rows will be retried and marked as `pending`.  
No duplicates will be created due to the `uniqueIdentifier` check.

---

## ✅ Boundary Mapping Process (Standardized)

---

### **1. During Initial Data Creation**

For each entity type in the creation process (before mapping occurs):

#### **a. During Project (Target) Creation**
- For each `PVar ID` in the sheet:
  - Create a mapping entry between `pvarId` and `boundaryCode`
  - `status = to_be_mapped`
  - `mappingId = null`
  - `type = "resource"`

---

#### **b. During Facility Creation**
- For each row:
  - Use `facilityName` as the `uniqueIdentifier`
  - Map `facilityName` with `boundaryCode` from the row
  - `status = to_be_mapped`
  - `mappingId = null`
  - `type = "facility"`

---

#### **c. During User Creation**
- For each user row:
  - Use `mobileNumber` as the `uniqueIdentifier`
  - Map `mobileNumber` with `boundaryCode`
  - `status = to_be_mapped`
  - `mappingId = null`
  - `type = "user"`

---

### **2. After Data Creation Completion**

Once all data (projects, facilities, users) is created:
- Use the same `referenceId` to retrieve rows from the mapping table.
- Separate the rows into two lists:
  - Rows requiring **mapping** → `status = to_be_mapped`
  - Rows requiring **detachment** → `status = to_be_detached`

---

### **3. Determine Rows to Detach**

To determine which rows need detachment:
- Compare the newly created data’s boundary codes with the old data’s boundary codes.
- If the new data does not contain a boundary that existed in the old data, the row should be marked for **detachment**.

---

### **4. Perform Mapping or Detachment**

#### **a. For `to_be_mapped` Rows**
- Process rows in batches (e.g., 100 at a time).
- Call the appropriate boundary mapping API.
- On successful mapping:
  - Update `status = mapped`
  - Set `mappingId` to the value returned by the mapping API.

---

#### **b. For `to_be_detached` Rows**
- Process rows in batches.
- Call the boundary detachment API.
- On successful detachment:
  - Update `status = detached`
  - Set `mappingId` to `null` or remove it.

---

### **5. Final Check**
- After processing all mapping and detachment batches:
  - Ensure no rows remain with `status = to_be_mapped` or `to_be_detached`.
  - If any rows remain, wait briefly and retry if needed.

---

### ✅ Mapping Table Structure Summary

- `uniqueIdentifier` → Mobile number, facility name, or PVar ID  
- `boundaryCode` → Boundary code associated with the boundary  
- `referenceId` → e.g., campaign ID  
- `status`: `to_be_mapped`, `mapped`, `to_be_detached`, `detached`  
- `mappingId`: Initially `null`, filled after the mapping API responds  
- `type`: `user`, `facility`, `resource`
