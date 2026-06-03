# 🔐 AWS VPC Security Architecture Project

## 📌 Project Overview

This project demonstrates the design and implementation of a secure AWS Virtual Private Cloud (VPC) following industry best practices for network segmentation, access control, and monitoring.

The infrastructure is designed to simulate a production-ready environment with multiple layers of security controls, including Security Groups, Network ACLs, Bastion Host access, VPC Flow Logs, and private/public subnet segregation.

---

## 🎯 Project Objectives

- Build a secure AWS network architecture.
- Implement defense-in-depth security controls.
- Restrict administrative access through a Bastion Host.
- Monitor network traffic using VPC Flow Logs.
- Apply least-privilege network access principles.
- Gain hands-on experience with AWS networking and cloud security.

---

## 🏗️ Architecture Components

### Networking
- Custom VPC
- Public Subnets
- Private Subnets
- Internet Gateway
- Route Tables
- NAT Gateway

### Security Controls
- Security Groups
- Network ACLs (NACLs)
- Bastion Host
- IAM Roles and Policies

### Monitoring & Logging
- VPC Flow Logs
- Amazon CloudWatch
- CloudWatch Log Groups

---

## 🛠️ AWS Services Used

- Amazon VPC
- Amazon EC2
- AWS IAM
- Amazon CloudWatch
- VPC Flow Logs
- NAT Gateway
- Internet Gateway

---

## 🔒 Security Features Implemented

### 1. Network Segmentation
- Public and Private subnet separation.
- Workloads isolated from direct internet access.

### 2. Bastion Host
- Administrative access only through Bastion Host.
- SSH restricted to authorized IP addresses.

### 3. Security Groups
- Stateful firewall protection.
- Only required ports are allowed.

Example:

| Service | Port | Access |
|----------|--------|---------|
| SSH | 22 | Authorized IP Only |
| HTTP | 80 | Internet |
| HTTPS | 443 | Internet |

### 4. Network ACLs
- Additional subnet-level filtering.
- Explicit inbound and outbound rules.

### 5. Traffic Monitoring
- VPC Flow Logs enabled.
- Logs sent to CloudWatch for analysis.

---

## 🚀 Deployment Steps

### Step 1
Create a custom VPC.

### Step 2
Create Public and Private Subnets.

### Step 3
Attach Internet Gateway.

### Step 4
Configure Route Tables.

### Step 5
Launch Bastion Host in Public Subnet.

### Step 6
Launch EC2 Instance in Private Subnet.

### Step 7
Configure Security Groups and NACLs.

### Step 8
Enable VPC Flow Logs.

### Step 9
Verify Connectivity and Logging.

---

## 🧪 Security Testing Performed

- Verified Bastion-only SSH access.
- Tested Security Group restrictions.
- Validated NACL filtering.
- Confirmed VPC Flow Log generation.
- Checked private subnet isolation.

---

## 📈 Learning Outcomes

Through this project, I gained practical experience in:

- AWS Networking
- Cloud Security Fundamentals
- Network Segmentation
- Security Groups & NACLs
- VPC Monitoring
- Bastion Host Security
- CloudWatch Logging
- Security Best Practices

---

## 💼 Skills Demonstrated

- AWS Cloud Security
- Network Security
- VPC Design
- Access Control
- Security Monitoring
- Infrastructure Security
- Cloud Operations


---

## ⭐ Future Enhancements

- AWS WAF Integration
- AWS Network Firewall
- Multi-AZ Deployment
- GuardDuty Threat Detection
