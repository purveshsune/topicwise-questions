# AWS Lambda Interview Q&A

### 1. What are AWS Lambda functions and how have you used them in your projects?

AWS Lambda is a **serverless compute service** — you run code without provisioning or managing servers.

In my projects, I’ve used Lambda for:
- **Image/file processing** triggered by S3 uploads
- **API backends** behind API Gateway
- **Data processing** from DynamoDB Streams
- **Automated deployments**, backups, and tag enforcement
- Glue logic between AWS services (event-driven automation)

---

### 2. Describe how you trigger Lambda functions using S3, EventBridge, or DynamoDB.

- **S3 Trigger**: When an object is uploaded to a bucket:
  
  S3 ➝ triggers Lambda

  Example: Resize images or scan files for malware

- **DynamoDB Streams**: Capture table change events and process them:

  DynamoDB ➝ Stream ➝ Lambda

  Example: Real-time analytics or sync data to another DB

- **EventBridge**: Event-driven triggers (cron, service events):

  EventBridge ➝ Lambda

  Example: Run nightly cleanup jobs or react to EC2 state changes

---

### 3. What are cold starts in Lambda, and how do you handle them?

**Cold Start**: Initial delay when Lambda spins up a new container. Happens when:
- Function is **invoked after idle time**
- New concurrent execution starts

**How to handle:**
- Use **provisioned concurrency** to pre-warm
- Keep code/lightweight and minimize dependencies
- Avoid VPC (unless needed) or use **VPC Lambda SnapStart**

---

### 4. How do you manage dependencies in your Lambda functions?

- **Python/Node.js**: Use `requirements.txt` or `package.json`
- Bundle via:
  - **Lambda Layers** (for shared deps across Lambdas)
  - **ZIP package** or **Docker image** deployment

For CI/CD:
- Package deps using **Docker or Terraform**
- Upload to S3 or deploy with tools like **Serverless Framework**, **SAM**, or **CDK**

---

### 5. How do you monitor and troubleshoot Lambda errors in production?

- **CloudWatch Logs** (automatically generated)
- Use `console.log()` / `print()` or structured logging
- Enable **AWS X-Ray** for tracing
- **CloudWatch Alarms** for function errors/throttles
- **Custom metrics** using `PutMetricData`

Real-world: We set alerts for `error count`, `duration`, and `invocation count`

---

### 6. What’s the difference between synchronous and asynchronous Lambda invocations?

| Type           | Behavior                                   | Examples                  |
|----------------|--------------------------------------------|---------------------------|
| Synchronous    | Waits for response                         | API Gateway, SDK calls    |
| Asynchronous   | Event queued and function is invoked later | S3, EventBridge, SNS      |

Synchronous = immediate response  
Async = decoupled, with retries and DLQs

---

### 7. How do you manage environment variables and secrets in Lambda?

- **Environment Variables** (set in config or Terraform)
  - Can mark as **encrypted (KMS)**

- **Secrets**:
  - Use **AWS Secrets Manager** or **SSM Parameter Store**
  - Access at runtime with SDK (boto3, AWS SDK)

Never hardcode secrets in code or env vars.

---

### 8. What is the maximum timeout and memory for Lambda, and how do you tune them?

| Resource | Max               |
|----------|------------------|
| Timeout  | 15 minutes (900s)|
| Memory   | 10 GB            |

You tune:
- **Timeout** based on job length
- **Memory** → affects CPU & network (scales together)

Tip: More memory = faster execution → sometimes **cheaper overall**

---

### 9. Have you implemented CI/CD pipelines to deploy Lambda functions? Describe the flow.

Yes, using **GitHub Actions**, **CodePipeline**, and **Terraform**.

Typical CI/CD flow:
1. Code pushed to GitHub
2. GitHub Actions lints, runs unit tests
3. Build + package Lambda (ZIP or Docker)
4. Deploy via:
   - Terraform (with remote state)
   - AWS SAM / Serverless Framework
5. Notify on success/failure (Slack/Email)

---

### 10. Can you share a real use case where you built or optimized a serverless app using Lambda?

Yes — here’s a real one:

**Use Case**: Automated invoice processing system  
**Flow**:
- Client uploads PDF to S3
- S3 triggers Lambda ➝ OCR process (Textract)
- Parsed data pushed to DynamoDB
- Another Lambda summarizes & emails report via SES
- All orchestrated with EventBridge

Optimizations:
- Split logic into **multiple smaller Lambdas** for modularity
- Used **provisioned concurrency** for better latency
- Reduced costs by scheduling **non-urgent Lambdas** with EventBridge rules