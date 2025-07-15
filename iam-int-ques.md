# ✅ IAM Interview Questions and Answers

---

###  1. How do you manage IAM users, roles, and groups in a DevOps environment?

In modern DevOps setups:
- **Users**: Rarely used directly. Typically used for CI/CD tools or limited human access.
- **Groups**: Used to organize human users (e.g., `DevOpsAdmins`, `ReadOnlyViewers`) and attach policies.
- **Roles**: Used **extensively** for service-to-service access (EC2, Lambda, CodeBuild, etc.) and **cross-account access**.

🎯 **Tools**:
- **Terraform** to create/manage IAM roles/policies
- **AWS SSO** for user management (instead of native IAM users)
- **Access Analyzer** & **IAM Access Advisor** for audits

---

###  2. What’s the difference between IAM roles and IAM users? When do you use each?

| Feature        | **IAM User**                | **IAM Role**                              |
|----------------|-----------------------------|--------------------------------------------|
| Login access   | Yes (username/password)     | No (assumed temporarily)                   |
| Use case       | Humans, legacy tools        | Services (EC2, Lambda, cross-account)      |
| Permanent creds| Yes (access keys)           | No (temporary STS tokens)                  |
| Best practice  | Limit or avoid              | Use for everything automated               |

📌 **Use IAM roles for:**
- EC2 instance profile
- Lambda execution role
- Federated user access
- Cross-account trust

---

###  3. How do you restrict permissions using IAM policies? Can you show an example?

IAM policies are JSON-based and define what actions are allowed or denied.

🔒 **Example: Allow only read-only access to S3**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

Add this to a role/user/group to grant limited access.

---

###  4. Have you implemented cross-account access using IAM roles? How?

Yes — here's how it's done:

- **Account A** (the *trusting* account) creates a role with a trust policy allowing **Account B** to assume it:

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::ACCOUNT_B_ID:root"
  },
  "Action": "sts:AssumeRole"
}
```

- **Account B** assumes the role using CLI or SDK:

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_A_ID:role/CrossAccountRole \
  --role-session-name DevOpsSession
```

✅ Used this for:
- CI/CD in Account B deploying infra in Account A
- Centralized monitoring/logging setups

---

###  5. How do you rotate and manage access keys securely?

- Avoid them when possible (use roles).
- For necessary keys:
  - Use **IAM Access Analyzer** to find unused/old keys.
  - Rotate every 90 days (AWS sends alerts).
  - Automate rotation with:
    - **Secrets Manager**
    - **AWS CLI** scripts
    - GitHub Actions + Terraform to rotate/store

✅ Pro Tip: **Never commit access keys to GitHub** — even in private repos. Use GitHub secrets for pipelines.

---

###  6. How do you grant temporary credentials to EC2 or Lambda without hardcoding?

Use **IAM Roles**:

- **EC2 Instance Profile**:
  ```bash
  curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
  ```

- **Lambda Execution Role**:
  Automatically injected into the Lambda runtime — SDKs like boto3 use it without config.

✅ All credentials are short-lived, rotated automatically.

---

###  7. How do you audit IAM usage and detect excessive privileges?

Tools:
- **IAM Access Analyzer** → Find unused permissions or resources
- **Access Advisor (IAM Console)** → See last used services per user/role
- **CloudTrail** + Athena → Deep audit on API calls
- **AWS Config** → Track config drift on IAM policies

✅ I used this combo to enforce least privilege by removing unused permissions periodically.

---

###  8. What tools or services have you used to analyze IAM policy risks?

- **AWS IAM Access Analyzer**
- **Cloudsplaining** (open source tool to detect IAM risk patterns)
- **PMapper** (for privilege escalation risk analysis)
- **ScoutSuite** or **PacBot** for broader cloud compliance scans

---

###  9. What’s the principle of least privilege and how do you enforce it?

> Only grant the **minimum permissions necessary** for a user/service to function.

How I enforce it:
- Use **managed policies** only as a starting point
- Replace with **inline policies** or **custom managed policies**
- Remove unused permissions periodically
- Split roles (read-only, writer, admin) instead of catch-all access

---

###  10. How have you used IAM conditions or policy variables for fine-grained access control?

IAM **conditions** let you scope permissions more tightly.

🔐 Example: Allow S3 write only in a specific folder (per user)

```json
"Condition": {
  "StringLike": {
    "s3:prefix": "home/${aws:username}/*"
  }
}
```

Also useful:
- **`aws:SourceIp`** → Allow only from corp VPN
- **`aws:RequestedRegion`** → Enforce region restrictions
- **`aws:PrincipalTag`** → Use tags to dynamically grant access

---