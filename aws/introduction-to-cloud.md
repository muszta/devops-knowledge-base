# 📘 Introduction to Cloud & AWS

## 📌 1. What is Cloud Computing?

**Cloud computing** means using computing resources (servers, storage, databases, networking) over the internet instead of owning and maintaining physical hardware.

A major provider is Amazon Web Services (AWS).

### 🔹 Benefits
- No upfront hardware costs  
- Scalability on demand  
- Pay-as-you-go pricing  
- High availability and reliability  
- Global infrastructure  

---

## 🧱 2. Cloud Service Models

### IaaS (Infrastructure as a Service)
- You manage: OS, applications  
- Provider manages: hardware, networking  
- Example: EC2  

### PaaS (Platform as a Service)
- You manage: applications  
- Provider manages: OS, runtime, infrastructure  
- Example: Elastic Beanstalk  

### SaaS (Software as a Service)
- Fully managed applications  
- Example: Gmail, Slack  

---

## 🌍 3. AWS Global Infrastructure

- **Region** = a physical geographic area (e.g. `eu-central-1`)  
- **Availability Zone (AZ)** = isolated data centers within a region  
- Designed for **fault tolerance and high availability**

---

## 🔐 4. Identity and Access Management (IAM)

AWS uses IAM to control access.

### 🔹 Key Concepts

- **IAM User** → represents a person or application  
- **IAM Role** → temporary permissions (recommended for apps)  
- **Policies** → JSON documents defining permissions  

### 🔹 Best Practices
- Never use root user for daily tasks  
- Use roles instead of long-term credentials  
- Apply least privilege principle  

---

## 💻 5. Core AWS Services

### 🖥️ Compute

- **EC2 (Elastic Compute Cloud)** → virtual servers  
- **Lambda** → serverless functions  

### 💾 Storage

- **S3 (Simple Storage Service)** → object storage  
- **EBS (Elastic Block Store)** → block storage for EC2  
- **EFS (Elastic File System)** → shared file storage  

### 🌐 Networking

- **VPC (Virtual Private Cloud)** → isolated network  
- **Route 53** → DNS service  
- **Load Balancer** → distributes traffic  

---

## 🧩 6. What is an AMI?

An **AMI (Amazon Machine Image)** is a template used to launch EC2 instances.

It includes:
- Operating system  
- Installed software  
- Configuration  

👉 Think of it like a **pre-configured server snapshot**

---

## 🔁 7. Infrastructure as Code (IaC)

Instead of manually creating resources, you define infrastructure in code.

### 🔹 Example Tool
- Terraform  

### 🔹 Benefits
- Version control  
- Reproducibility  
- Automation  

---

## 🔄 8. Basic Cloud Architecture Flow

1. User sends request  
2. DNS resolves via Route 53  
3. Traffic hits Load Balancer  
4. Request goes to EC2 / application  
5. Data stored in S3 / database  

---

## 📦 9. Shared Responsibility Model

| Responsibility | AWS | You |
|------|------|------|
| Hardware | ✅ | ❌ |
| Networking | ✅ | ❌ |
| OS Patching | ❌ | ✅ |
| Application Security | ❌ | ✅ |

👉 AWS secures the **cloud**, you secure what’s **in the cloud**

---

## 🚀 10. Summary

- Cloud = on-demand infrastructure  
- AWS = leading cloud provider  
- Use IAM for secure access  
- Core services: EC2, S3, VPC  
- Use IaC (Terraform) for automation  
- Always follow best practices (least privilege, high availability)