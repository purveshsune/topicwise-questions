#  Dynamodb Interview Questions and Answers

---

### ✅ **Q1: What is DynamoDB and how have you used it in your AWS DevOps projects?**

**Answer:**  
Amazon DynamoDB is a fully managed NoSQL database that offers single-digit millisecond latency at any scale. I’ve used it in DevOps projects to:
- Store configuration and metadata for serverless applications.
- Maintain state in event-driven architectures.
- Store user sessions and auth tokens in web apps.
- Manage Terraform state locks in conjunction with S3.
- Capture and audit infrastructure provisioning workflows.

---

### ✅ **Q2: How do you design partition keys and sort keys for efficient query performance?**

**Answer:**  
Efficient design is crucial to avoid hotspots and throttling:
- I select a **partition key** that ensures even data distribution (e.g., user ID, tenant ID).
- If query filtering is needed, I add a **sort key** (e.g., timestamp, event type).
- I use **composite keys** to support time-series queries.
- For high-volume writes, I sometimes use random suffixes or prefixes to distribute load.

Example: For IoT logs, I used `device_id` as the partition key and `timestamp` as the sort key.

---

### ✅ **Q3: Have you worked with DynamoDB Streams? If yes, how did you use them?**

**Answer:**  
Yes. DynamoDB Streams capture changes in the table (insert, modify, delete) in near real-time.

Use cases:
- Triggered **AWS Lambda** to sync data changes to another service (e.g., Elasticsearch or S3).
- Used Streams + Lambda for **event-driven workflows** (like sending emails on user updates).
- Created a **CDC (change data capture)** pipeline to maintain audit logs.

---

### ✅ **Q4: How do you implement backup and restore in DynamoDB?**

**Answer:**  
I’ve used both:
- **Point-in-time recovery (PITR)**: Enabled for continuous backups (up to 35 days).
- **On-demand backups**: Used in CI/CD pipelines before schema migrations.

For restores:
- Used **AWS Console**, **CLI**, or **CloudFormation** to restore tables to a specific timestamp or into new tables for verification.

I also automate periodic backups using Lambda with scheduled CloudWatch events.

---

### ✅ **Q5: How does DynamoDB handle read/write capacity, and how do you choose between provisioned and on-demand modes?**

**Answer:**  
**Modes:**
- **Provisioned**: I define `read/write capacity units (RCUs/WCUs)` manually.
- **On-demand**: DynamoDB auto-scales capacity as needed (pay-per-request).

**Choice criteria:**
- Use **on-demand** for unpredictable or low-traffic workloads.
- Use **provisioned** with **auto-scaling** for steady workloads with predictable usage.

In one project, we started with on-demand, then moved to provisioned with auto-scaling after identifying stable traffic patterns.

---

### ✅ **Q6: Can you explain how TTL (Time to Live) works in DynamoDB and when you’ve used it?**

**Answer:**  
**TTL** allows automatic deletion of expired items based on a timestamp attribute.

Use cases:
- Expire user sessions or auth tokens.
- Clean up temporary or cache-like data (e.g., email verification links).
- Manage cleanup of logs older than X days.

It's cost-efficient and reduces manual housekeeping.

---

### ✅ **Q7: Have you used DynamoDB in a serverless application? How did you integrate it with Lambda?**

**Answer:**  
Yes, several times. I integrated DynamoDB by:
- Using the **AWS SDK (Boto3, AWS SDK for Node.js, etc.)** inside Lambda to query or write data.
- Assigning **IAM roles** to Lambda with least-privilege access to specific tables.
- Triggering Lambdas via **DynamoDB Streams** for reactive processing.

Example: In a chat app, messages were stored in DynamoDB and each new message triggered a Lambda to notify subscribers via WebSocket.

---

### ✅ **Q8: What are some best practices you follow to optimize DynamoDB performance?**

**Answer:**  
- **Design for access patterns** (read-optimized vs write-heavy).
- Use **secondary indexes** (GSI/LSI) carefully to support multiple query patterns.
- Use **batch operations** for reads/writes to reduce round trips.
- Avoid large item sizes and hot partitions.
- Monitor with **CloudWatch metrics** (e.g., throttles, consumed capacity).
- Enable **DAX** (DynamoDB Accelerator) if low-latency reads are critical.

---

### ✅ **Q9: How do you secure DynamoDB access across multiple Lambda functions or services?**

**Answer:**  
- Use **IAM roles with scoped permissions** (`dynamodb:GetItem`, `dynamodb:PutItem`, etc.).
- Use **resource-based policies** for cross-account access if needed.
- Apply **least privilege** principle — different Lambdas get different roles with specific access.
- Implement **VPC endpoints** to keep traffic private if using VPC-based Lambdas.

---

### ✅ **Q10: Describe a scenario where you had to troubleshoot or debug a DynamoDB performance issue.**

**Answer:**  
In one case, a Lambda function was intermittently throttled when writing to DynamoDB.

**Root cause:** A high volume of writes was targeting a single partition key due to poor design.

**Fix:**
- Revisited key design to distribute writes more evenly (added random suffix to partition key).
- Enabled **auto-scaling** on write capacity.
- Added **retry logic** with exponential backoff in Lambda.

Also used **CloudWatch dashboards** and **DynamoDB Insights** to monitor and fine-tune throughput.

---