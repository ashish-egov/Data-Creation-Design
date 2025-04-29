### ✅ Final Flow

1. **Input**  
   - You upload a sheet.
   - You also provide a `reference_id` (e.g., `campaignId = "cmp-123"`).

2. **Insert New Rows**  
   - For each row in the sheet:
     - Generate a `unique_identifier` from the row (e.g., mobile number).
     - Check if it already exists in `data_creation_status` for that `reference_id`.
     - If not, insert it with:
       - `status = 'pending'`
       - `type` = `"campaign"` (or appropriate type)
       - `reference_id = 'cmp-123'`
       - `row_data` = full row JSON

3. **Mark Failed Rows as Pending Again**  
   - For the same `reference_id`, update all rows where `status = 'failed'` → `'pending'`

4. **Send to Kafka in Batches**  
   - Pick 100 rows with `status = 'pending'` for that `reference_id`
   - Send to Kafka topic

5. **Kafka Consumer Processing**  
   - Consume the batch
   - Create actual records (e.g., campaign targets)
   - If successful, update:
     - `status = 'success'`
     - (optional) update back `reference_id` if needed

6. **Final Validation**  
   - After all batches, check:
     - All rows for that `reference_id` should have `status = 'success'`
