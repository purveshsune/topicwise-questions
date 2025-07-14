
# Ansible Interview Q&A

---

### Q1. **What is Ansible and how have you used it in your AWS projects?**

**Answer**:  
Ansible is an open-source automation tool used for configuration management, application deployment, and orchestration. In my AWS projects, I’ve used Ansible primarily to automate provisioning and configuration of EC2 instances, install dependencies, deploy applications, and manage infrastructure state. I’ve also used Ansible Tower for visual workflows and RBAC.

---

### Q2. **Can you explain the difference between an Ansible playbook, role, and inventory? How have you structured them in your projects?**

**Answer**:  
- **Playbook**: A YAML file that defines tasks to run on remote hosts.
- **Role**: A modular structure in Ansible for organizing playbooks, variables, handlers, files, and templates.
- **Inventory**: A file or script that defines which hosts Ansible should target.

In my projects, I structure them like:
```
inventory/
  - dev.ini
  - prod.ini
roles/
  - webserver/
    - tasks/
    - templates/
    - defaults/
playbooks/
  - site.yml
```
This structure helps with reuse and scalability across environments.

---

### Q3. **How do you use Ansible to configure EC2 instances in AWS?**

**Answer**:  
I use Ansible to:
- Connect to EC2 instances via SSH using a key pair.
- Use roles to install software (like NGINX, Docker, etc.).
- Push configuration files using templates.
- Tag resources and manage security groups if needed via `boto` modules.
Sometimes, I integrate this with Terraform to provision the EC2s first, then use Ansible for configuration.

---

### Q4. **Have you used Ansible with dynamic inventories, especially in AWS? How do they work?**

**Answer**:  
Yes, I’ve used the `aws_ec2` dynamic inventory plugin. It uses AWS APIs to fetch real-time information about EC2 instances based on filters or tags.

Steps I follow:
1. Configure `aws_ec2.yaml` with AWS credentials and filters.
2. Run playbooks using `-i aws_ec2.yaml`.
3. Use tags to group instances dynamically.

This avoids hardcoding IPs and keeps the inventory flexible as instances scale.

---

### Q5. **Walk me through a real scenario where you used Ansible to automate infrastructure or application deployment.**

**Answer**:  
I automated the deployment of a Node.js app on an auto-scaled EC2 fleet:
- Terraform created the VPC, ALB, and EC2 launch templates.
- Ansible configured EC2 instances (Node.js, PM2, app code deployment).
- Handled NGINX config via templates.
- Integrated with CodePipeline to trigger Ansible via Jenkins on new commits.

This reduced manual setup time from hours to minutes and made the deployment consistent and repeatable.

---

### Q6. **How do you manage secrets and sensitive data (like AWS credentials or passwords) in Ansible?**

**Answer**:  
- Used **Ansible Vault** to encrypt secrets like DB passwords and API keys.
- Stored AWS credentials securely in environment variables or IAM roles (on EC2).
- In some projects, used **HashiCorp Vault** integrated with Ansible for dynamic secrets.

Example command:
```bash
ansible-vault encrypt secrets.yml
```

---

### Q7. **How do you ensure idempotency in your playbooks? Can you give an example?**

**Answer**:  
Idempotency means running a playbook multiple times won’t change the system after the first run unless something has changed.

**Example**:
```yaml
- name: Install NGINX
  apt:
    name: nginx
    state: present
```
Even if you run this 10 times, NGINX will only be installed once. I avoid using shell/command modules unless necessary and use modules like `apt`, `copy`, `template` that support idempotency.

---

### Q8. **How do you handle error handling and retries in Ansible tasks?**

**Answer**:  
- Use `retries` and `delay`:
```yaml
- name: Wait for service to be available
  uri:
    url: http://localhost:3000/health
    status_code: 200
  register: result
  retries: 5
  delay: 10
  until: result.status == 200
```
- Use `ignore_errors: yes` for non-critical steps.
- Group related tasks with `block` and use `rescue` and `always` for structured error handling.

---

### Q9. **Have you integrated Ansible with Jenkins or any CI/CD tool? If yes, describe the flow.**

**Answer**:  
Yes. In a CI/CD pipeline with Jenkins:
- Jenkins pulled code from GitHub on a push.
- Ran test suites.
- On success, triggered Ansible to deploy to staging or production.
- Ansible pulled inventory from AWS using dynamic inventory.
- Deployment tasks included pulling the latest Docker image, updating services, and notifying Slack.

---

### Q10. **What’s the most complex Ansible project you’ve worked on, and what were the key challenges?**

**Answer**:  
The most complex project was a hybrid cloud deployment for a microservices-based app across AWS and on-prem servers.  
Challenges:
- Managing inventories for multiple environments.
- Coordinating service dependencies (DBs had to be ready before apps).
- Securely managing secrets across environments.
- Ensuring idempotency and rollback in case of partial failures.

I overcame these with:
- Modular roles per service.
- Using Ansible Tower for better visibility and RBAC.
- Playbooks with conditional checks and retries.

---
