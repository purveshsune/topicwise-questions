#  Cloudwatch Interview Questions and Answers

---

### **1. What is Amazon CloudWatch and how do you use it in DevOps?**

**Amazon CloudWatch** is a monitoring and observability service designed for AWS resources and applications. In DevOps projects, it’s used to:
- Monitor system performance (EC2, Lambda, containers, etc.)
- Set alarms on thresholds
- Collect and analyze logs
- Create dashboards for real-time visibility
- Automate responses using EventBridge (formerly CloudWatch Events)

---

### **2. How do you monitor EC2, Lambda, and containerized workloads using CloudWatch?**

- **EC2**: 
  - Out-of-the-box metrics: CPU, disk, network.
  - Install **CloudWatch Agent** to monitor memory, disk space, and custom metrics.
  - Push logs to CloudWatch Logs (e.g., `/var/log/syslog`).

- **Lambda**: 
  - Auto-publishes logs and metrics (invocations, errors, duration).
  - Use **CloudWatch Insights** to search logs (e.g., `REPORT` lines for duration/memory).

- **Containers (ECS/EKS)**:
  - Enable **Container Insights** for metrics like CPU/memory per container.
  - Use Fluent Bit or CloudWatch Agent in a sidecar pattern to ship logs.

---

### **3. Difference between CloudWatch Logs, Metrics, and Events?**

| Feature | Description |
|--------|-------------|
| **Logs** | Time-stamped logs from apps, Lambda, EC2, etc. Can be searched and analyzed. |
| **Metrics** | Numerical time series (e.g., CPU %, request count). Trigger alarms. |
| **Events (EventBridge)** | Rules based on events from AWS services (e.g., EC2 state change). Useful for automation. |

---

### **4. How do you create custom CloudWatch metrics? Use case?**

You can publish metrics using AWS SDKs or the CloudWatch Agent.

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "UserSignupCount" \
  --value 1
```

**Use case**: Track business KPIs (e.g., orders processed per hour) or custom app health checks.

---

### **5. How do you configure and manage CloudWatch Alarms?**

- Go to CloudWatch → Alarms → Create Alarm.
- Choose a metric (e.g., CPUUtilization > 80% for 5 mins).
- Set up notifications (e.g., SNS to Slack/email).
- Optionally automate actions (e.g., reboot instance or scale out).

Use **composite alarms** to combine multiple conditions (e.g., high CPU + low disk).

---

### **6. Have you used CloudWatch dashboards? What widgets or metrics have you included?**

Yes—dashboards give a real-time visual view of system health.

**Common widgets/metrics**:
- Line graphs for EC2 CPU/RAM
- Lambda invocation duration/errors
- ECS CPU/Memory per service
- Custom KPIs (e.g., API latency)
- Log query results from Insights

---

### **7. How do you ship application logs from containers or EC2 to CloudWatch Logs?**

**For EC2**:
- Install **CloudWatch Agent** or **CloudWatch Logs Agent**.
- Configure `/etc/cloudwatch-agent/config.json` to watch log files.

**For containers**:
- Use **Fluent Bit**, AWS FireLens, or CloudWatch Agent in a sidecar.
- Mount the log file directory and route logs to CloudWatch.

---

### **8. What is CloudWatch Agent and how do you install and configure it?**

**CloudWatch Agent** collects:
- System-level metrics (memory, disk, etc.)
- Application logs

**Install**:
```bash
sudo yum install amazon-cloudwatch-agent
```

**Configure**:
Use `amazon-cloudwatch-agent-config-wizard` or provide a JSON file.

**Start agent**:
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -c file:/path/to/config.json -s
```

---

### **9. How do you analyze logs in CloudWatch Logs Insights?**

Go to **CloudWatch Logs → Insights**, select a log group, and run SQL-like queries.

**Examples**:
```sql
fields @timestamp, @message
| filter @message like /error/
| sort @timestamp desc
| limit 20
```

Great for troubleshooting latency, error spikes, or custom app logs.

---

### **10. Describe a real incident you detected and mitigated using CloudWatch.**

**Scenario**: An app was experiencing slow response times during peak hours.

**Detection**:
- CloudWatch Alarm on API Gateway latency.
- Dashboards showed Lambda duration spiking.

**Investigation**:
- Logs Insights revealed long DB query times.
- Correlated metrics showed RDS CPU at 90%.

**Mitigation**:
- Increased RDS instance size.
- Added query caching.
- Created CloudWatch Alarm on RDS CPU > 80%.

---