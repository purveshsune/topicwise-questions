# ✅ EC2 Interview Questions and Answers


###  Q1: What are some of the key EC2 instance types you’ve worked with, and how do you choose the right one?

**Answer:**  
I’ve worked with several EC2 instance types:
- **t3/t4g**: Cost-efficient general-purpose for small workloads.
- **m5/m6i**: Balanced compute/memory for app servers and Jenkins agents.
- **c5/c6g**: Compute-optimized for CI workloads or build servers.
- **r5/r6g**: Memory-optimized for caching layers and in-memory databases.
- **p3/g4**: For ML workloads and GPU-intensive tasks.

**Selection factors:**
- **Workload type**: Compute vs memory vs storage intensive.
- **Burstable vs consistent performance**
- **Cost optimization** with Spot Instances or Graviton (ARM-based) where possible.
- **Compatibility** with existing software stacks.

---

###  Q2: How do you automate EC2 instance provisioning using Terraform or CloudFormation?

**Answer:**  
In **Terraform**, I use the `aws_instance` or `aws_launch_template` resource:
- Define AMI, instance type, key pair, tags, security groups, and user data.
- Attach IAM roles, EBS volumes, and monitoring settings.
- Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
  key_name      = "team-key"
  user_data     = file("init.sh")
  tags = {
    Name = "web-server"
  }
}
```

In **CloudFormation**, similar configuration via `AWS::EC2::Instance`.

---

###  Q3: How do you manage EC2 SSH key pairs across teams securely?

**Answer:**
- Avoid sharing private keys — instead use **AWS Systems Manager (SSM)** for session access.
- Use **temporary IAM roles** + **Session Manager** as an SSH alternative (no keys needed).
- For key-based access:
  - Use **per-team or per-user key pairs**.
  - Rotate keys regularly.
  - Store in **Secrets Manager** or **SSM Parameter Store** securely.
- Manage public keys via automation in launch templates or user data scripts.

---

###  Q4: What is a user data script in EC2 and how have you used it?

**Answer:**
A **user data script** is a shell script that runs **once at instance launch**. I’ve used it to:
- Install packages (e.g., NGINX, Docker).
- Configure app settings.
- Register instances with load balancers.
- Pull code from GitHub/S3.
- Run configuration management tools like Ansible or Chef.

Example snippet:
```bash
#!/bin/bash
yum update -y
amazon-linux-extras install nginx1 -y
systemctl enable nginx && systemctl start nginx
```

---

###  Q5: How do you enable auto-scaling for EC2 instances?

**Answer:**
- Set up an **Auto Scaling Group (ASG)** with a **Launch Template** or **Launch Configuration**.
- Define:
  - **Min/max/desired capacity**
  - **Scaling policies** (CPU, memory, ALB request count, custom metrics via CloudWatch)
- Attach the ASG to a **Load Balancer** for health checks and traffic distribution.
- Use Terraform (`aws_autoscaling_group`) or CloudFormation (`AWS::AutoScaling::AutoScalingGroup`) for automation.

---

### Q6: Describe how you would troubleshoot an EC2 instance that is not reachable via SSH.

**Answer:**
1. **Check instance state** (running or stopped?).
2. **Verify security groups**: Port 22 open to allowed IPs.
3. **Check NACLs**: Ensure inbound/outbound rules are correct.
4. **Check route table**: Proper route to IGW if public instance.
5. **Validate public IP**: Make sure it has one if needed.
6. **Check key pair**: Right key? Permissions correct?
7. **Use EC2 Serial Console or SSM Session Manager** for deeper debugging.

---

###  Q7: How do you manage updates and patching across EC2 instances?

**Answer:**
- Use **AWS Systems Manager Patch Manager** with patch baselines and maintenance windows.
- Or run update scripts in cron, or via **user data at launch time**.
- Monitor patch compliance with **SSM inventory and compliance reports**.
- For immutable infrastructure, replace instead of patch (rebuild AMIs via Packer or EC2 Image Builder).

---

###  Q8: What is the difference between an EBS volume and an instance store?

**Answer:**
| Feature | **EBS Volume** | **Instance Store** |
|--------|----------------|---------------------|
| Persistent? | ✅ Yes | ❌ No (ephemeral) |
| Data after stop/reboot? | ✅ Retained | ❌ Lost |
| Performance | High | Extremely fast (NVMe/SSD) |
| Use cases | Boot volumes, databases | Temp files, buffer/cache |

I mostly use **EBS** for production. **Instance store** is useful for high-speed scratch storage.

---

###  Q9: How do you configure EC2 instance monitoring and alerts?

**Answer:**
- Enable **CloudWatch detailed monitoring** (1-min intervals).
- Set up **alarms** on:
  - CPU, memory, disk (memory/disk via CloudWatch Agent)
  - Status checks (system and instance)
- Use **SNS** to notify via email, Slack, or Ops tools.
- Install **CloudWatch Agent** to push OS-level metrics/logs.

---

###  Q10: How have you integrated EC2 instances with Jenkins agents or other CI/CD tools?

**Answer:**
- Created **ephemeral Jenkins agents** using EC2 Spot Instances via EC2 plugin.
- Used **Launch Templates** to spin agents with required tools using user data scripts.
- Managed IAM roles for least-privileged access.
- For Docker-based builds, installed Docker and mounted EBS volumes for caching.
- Integrated Jenkins with **SSM Session Manager** for secure command execution.

---
