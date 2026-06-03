# 🔐 AWS VPC Network Security Project

A production-grade, 3-tier AWS VPC built with layered network security — designed to demonstrate real-world cloud infrastructure and security hardening skills.

---

## 📌 Project Summary

This project involves designing and deploying a secure AWS Virtual Private Cloud (VPC) from scratch, implementing multiple layers of network security including Security Groups, Network ACLs, a hardened Bastion Host, and full traffic monitoring via VPC Flow Logs and CloudWatch.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS CLOUD — prod-vpc                         │
│                          CIDR: 10.0.0.0/16                          │
│                                                                     │
│    🌐 Internet                                                      │
│         │                                                           │
│    ┌────▼────────────────────────────────────────────────────┐     │
│    │               Internet Gateway (IGW)                    │     │
│    └────┬────────────────────────────────────────────────────┘     │
│         │                                                           │
│    ┌────▼────────────────────────────────────────────────────┐     │
│    │          PUBLIC SUBNET — 10.0.1.0/24                    │     │
│    │                                                         │     │
│    │   ┌─────────────────────┐   ┌──────────────────────┐   │     │
│    │   │  🛡️  Bastion Host   │   │    NAT Gateway  📮   │   │     │
│    │   │   EC2 t2.micro      │   │  Outbound only        │   │     │
│    │   │   SSH port 22       │   │  for private subnet   │   │     │
│    │   │   Your IP only      │   │                       │   │     │
│    │   └──────────┬──────────┘   └──────────────────────┘   │     │
│    └──────────────│─────────────────────────────────────────┘     │
│                   │ SSH hop (bastion only)                          │
│    ┌──────────────▼──────────────────────────────────────────┐     │
│    │          PRIVATE SUBNET — 10.0.2.0/24                   │     │
│    │   ┌──────────────────────────────────────────────────┐  │     │
│    │   │  💻  App Server (EC2)                            │  │     │
│    │   │  No public IP — zero direct internet exposure    │  │     │
│    │   │  Outbound via NAT Gateway only                   │  │     │
│    │   └──────────────────────┬───────────────────────────┘  │     │
│    └─────────────────────────│───────────────────────────────┘     │
│                               │ MySQL port 3306 only                │
│    ┌─────────────────────────▼───────────────────────────────┐     │
│    │          DATABASE SUBNET — 10.0.3.0/24                  │     │
│    │   ┌──────────────────────────────────────────────────┐  │     │
│    │   │  🗄️  Database (RDS)                              │  │     │
│    │   │  No internet route — completely isolated          │  │     │
│    │   │  Accessible only from private subnet             │  │     │
│    │   └──────────────────────────────────────────────────┘  │     │
│    └─────────────────────────────────────────────────────────┘     │
│                                                                     │
│    📹 VPC Flow Logs ──────────────────────► CloudWatch Logs        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ☁️ AWS Services Used

| Service            | Role in Project                                          |
|--------------------|----------------------------------------------------------|
| VPC                | Isolated private network — prod-vpc (10.0.0.0/16)       |
| Subnets            | 3-tier separation — public, private, database            |
| Internet Gateway   | Controlled entry and exit point from the internet        |
| NAT Gateway        | Outbound-only internet access for private subnet         |
| Route Tables       | Per-subnet traffic routing rules                         |
| Security Groups    | Stateful, instance-level firewall                        |
| Network ACLs       | Stateless, subnet-level firewall                         |
| EC2                | Bastion host (public) and app server (private)           |
| VPC Flow Logs      | Full network traffic metadata capture                    |
| CloudWatch Logs    | Centralised log storage and Insights querying            |
| IAM                | Service role granting Flow Logs permission to CloudWatch |

---

## 🔒 Security Layers Implemented

### Layer 1 — Subnet Isolation

Three subnets with strict separation:

- **Public subnet** — internet-facing, bastion host only
- **Private subnet** — no direct internet access, app servers live here
- **DB subnet** — zero internet route, database only, fully isolated

### Layer 2 — Route Table Hardening

| Route Table  | Destination  | Target           | Purpose                        |
|--------------|--------------|------------------|--------------------------------|
| public-rt    | 10.0.0.0/16  | local            | VPC internal traffic           |
| public-rt    | 0.0.0.0/0    | Internet Gateway | Full internet access           |
| private-rt   | 10.0.0.0/16  | local            | VPC internal traffic           |
| private-rt   | 0.0.0.0/0    | NAT Gateway      | Outbound only — no inbound     |
| db-rt        | 10.0.0.0/16  | local            | VPC internal only — no internet|

### Layer 3 — Security Groups (Stateful — instance level)

| Security Group  | Inbound Port | Source              | Outbound       |
|-----------------|--------------|---------------------|----------------|
| bastion-sg      | 22 (SSH)     | My IP only /32      | All traffic    |
| private-ec2-sg  | 22 (SSH)     | bastion-sg ID       | HTTPS 443 only |
| db-sg           | 3306 (MySQL) | private-ec2-sg ID   | None           |

> **Key pattern used:** Security Group ID as source — not an IP address. Access is controlled by group membership, so even if the bastion IP changes, the rule stays valid automatically. This is the production-grade pattern.

### Layer 4 — Network ACL (Stateless — subnet level)

Applied to: `private-subnet-1a`

**Inbound Rules**

| Rule No. | Protocol | Port Range    | Source        | Action |
|----------|----------|---------------|---------------|--------|
| 100      | TCP      | 22 (SSH)      | 10.0.1.0/24   | ALLOW  |
| 200      | TCP      | 1024 – 65535  | 0.0.0.0/0     | ALLOW  |
| *        | All      | All           | 0.0.0.0/0     | DENY   |

**Outbound Rules**

| Rule No. | Protocol | Port Range    | Destination   | Action |
|----------|----------|---------------|---------------|--------|
| 100      | TCP      | 443 (HTTPS)   | 0.0.0.0/0     | ALLOW  |
| 200      | TCP      | 1024 – 65535  | 0.0.0.0/0     | ALLOW  |
| *        | All      | All           | 0.0.0.0/0     | DENY   |

> **Why ports 1024–65535?** NACLs are stateless — they have no memory of connections. When the app server sends a request out, the response comes back on a random high port (ephemeral port). Rule 200 explicitly allows these return packets. Security Groups handle this automatically — NACLs do not.

### Layer 5 — VPC Flow Logs and Monitoring

| Setting        | Value                        |
|----------------|------------------------------|
| Filter         | All (ACCEPT + REJECT)        |
| Destination    | CloudWatch Logs              |
| Log Group      | /vpc/prod-flow-logs          |
| IAM Role       | Auto-created by AWS          |

CloudWatch Insights query to detect blocked/suspicious traffic:

```sql
fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter action = "REJECT"
| sort @timestamp desc
| limit 20
```

---

## 🛡️ Bastion Host Design

The bastion host is the **single controlled entry point** into the private network.

| Property          | Value                                          |
|-------------------|------------------------------------------------|
| Location          | Public subnet (has public IP)                  |
| Instance type     | EC2 t2.micro (free tier)                       |
| SSH access        | Restricted to my IP only using /32 CIDR        |
| Security group    | bastion-sg — port 22 from my IP only           |
| Purpose           | Jump server — hop into private instances       |

Private app server has **no public IP at all** — it is invisible to the internet. The only path in is through the bastion.

```bash
# Step 1 — SSH into bastion host
ssh -i bastion-key.pem ec2-user@<bastion-public-ip>

# Step 2 — From inside bastion, hop into private app server
ssh -i bastion-key.pem ec2-user@<private-instance-private-ip>
```

---

## 🗺️ CIDR Plan

| Resource           | CIDR Block      | Total IPs | Usage                        |
|--------------------|-----------------|-----------|------------------------------|
| VPC (prod-vpc)     | 10.0.0.0/16     | 65,536    | Entire private network       |
| Public subnet      | 10.0.1.0/24     | 256       | Bastion host, NAT Gateway    |
| Private subnet     | 10.0.2.0/24     | 256       | App servers                  |
| DB subnet          | 10.0.3.0/24     | 256       | Database instances           |
| My IP (SSH rule)   | x.x.x.x/32     | 1         | Only my laptop can SSH in    |

---

## ✅ Key Learnings

- Difference between **stateful (Security Group)** and **stateless (NACL)** firewalls — and why both are needed together
- Why **ephemeral ports 1024–65535** must be explicitly allowed in NACL rules for return traffic
- Using **Security Group ID as source** instead of IP address — production-grade least-privilege pattern
- Why **DB subnet has no internet route** — no route means no network path, full isolation regardless of other configs
- How **VPC Flow Logs** capture metadata (not payload) for every network connection — ACCEPT and REJECT
- **Bastion host pattern** — single SSH entry point, private servers have zero public IP exposure

---

## 📁 Project Structure

```
aws-vpc-network-security/
│
├── README.md
└── screenshots/
    ├── 01-vpc-created.png
    ├── 02-three-subnets.png
    ├── 03-igw-attached.png
    ├── 04-public-route-table.png
    ├── 05-all-route-tables.png
    ├── 06-bastion-sg-rules.png
    ├── 07-private-sg-source.png
    ├── 08-all-security-groups.png
    ├── 09-nacl-rules.png
    ├── 10-flow-log-active.png
    ├── 11-ssh-into-bastion.png
    └── 12-cloudwatch-rejects.png
```

---

## 👩‍💻 Author

**Krinakshi**
Junior Cloud Engineer | Linux Administrator
B.Tech Computer Science — Cloud Computing Specialisation

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
