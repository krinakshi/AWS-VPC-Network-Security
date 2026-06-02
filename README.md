# AWS-VPC-Network-Security
Production-grade 3-tier AWS VPC with Security Groups, NACLs, Bastion Host and VPC Flow Logs

🔐 AWS VPC Network Security Project
A production-grade, 3-tier AWS VPC built with layered network security — designed to demonstrate real-world cloud infrastructure and security hardening skills.

📌 Project Summary
This project involves designing and deploying a secure AWS Virtual Private Cloud (VPC) from scratch, implementing multiple layers of network security including Security Groups, Network ACLs, a hardened Bastion Host, and full traffic monitoring via VPC Flow Logs and CloudWatch.

🏗️ Architecture Overview
                         🌐 Internet
                              |
                    [ Internet Gateway ]
                              |
          ┌──────────────────────────────────────┐
          │         PUBLIC SUBNET (10.0.1.0/24)  │
          │   🛡️  Bastion Host (EC2 t2.micro)    │
          │       SSH — port 22 — your IP only   │
          └──────────────┬───────────────────────┘
                         │ SSH via bastion only
          ┌──────────────▼───────────────────────┐
          │        PRIVATE SUBNET (10.0.2.0/24)  │
          │   💻  App Server (EC2)               │
          │       No public IP — zero exposure   │
          │       Outbound via NAT Gateway only  │
          └──────────────┬───────────────────────┘
                         │ Port 3306 only
          ┌──────────────▼───────────────────────┐
          │          DB SUBNET (10.0.3.0/24)     │
          │   🗄️  Database (RDS)                 │
          │       No internet route at all       │
          │       Fully isolated from outside    │
          └──────────────────────────────────────┘
                         │
              📹 VPC Flow Logs → CloudWatch

☁️ AWS Services Used
ServicePurposeVPCIsolated private network (prod-vpc, CIDR 10.0.0.0/16)Subnets3-tier separation — public, private, databaseInternet GatewayControlled entry point from the internetNAT GatewayOutbound-only internet access for private subnetRoute TablesTraffic routing rules per subnetSecurity GroupsStateful, instance-level firewallNetwork ACLsStateless, subnet-level firewallEC2Bastion host and app server instancesVPC Flow LogsFull traffic metadata captureCloudWatch LogsCentralised log storage and queryingIAMService role for Flow Logs permissions

🔒 Security Layers Implemented
Layer 1 — Subnet Isolation
Three subnets with strict separation:

Public subnet — internet-facing, bastion host only
Private subnet — no direct internet access, app servers
DB subnet — zero internet route, database only

Layer 2 — Route Table Hardening

Public route table → 0.0.0.0/0 points to Internet Gateway
Private route table → 0.0.0.0/0 points to NAT Gateway (outbound only)
DB route table → no internet route at all

Layer 3 — Security Groups (Stateful)
Security GroupInbound RuleSourcebastion-sgSSH port 22My IP only (x.x.x.x/32)private-ec2-sgSSH port 22bastion-sg ID (not IP)db-sgMySQL port 3306private-ec2-sg ID

Key pattern: Security Group ID used as source — not an IP address. This means access is controlled by group membership, not by IP, making it robust even if the bastion IP changes.

Layer 4 — Network ACL (Stateless)
Applied to private subnet:
Inbound rules:
RulePortSourceAction10022 (SSH)10.0.1.0/24ALLOW2001024–65535 (ephemeral)0.0.0.0/0ALLOW*AllAllDENY
Outbound rules:
RulePortDestinationAction100443 (HTTPS)0.0.0.0/0ALLOW2001024–65535 (ephemeral)0.0.0.0/0ALLOW*AllAllDENY

Why ephemeral ports? NACLs are stateless — they do not remember connections. Ports 1024–65535 must be explicitly allowed for return traffic, unlike Security Groups which handle this automatically.

Layer 5 — VPC Flow Logs + Monitoring

Flow Logs enabled on the entire VPC (Filter: All)
Logs shipped to CloudWatch Log Group: /vpc/prod-flow-logs
CloudWatch Insights query to detect blocked traffic:

sqlfields @timestamp, srcAddr, dstAddr, dstPort, action
| filter action = "REJECT"
| sort @timestamp desc
| limit 20

🛡️ Bastion Host Design
The bastion host is the single controlled entry point into the private network:

Deployed in the public subnet with a public IP
SSH access restricted to my IP only using /32 CIDR
Private app server has no public IP — invisible to internet
Access flow: My laptop → Bastion (SSH) → App Server (SSH)
Private server security group allows SSH only from bastion-sg — not from any IP

bash# Step 1 — SSH into bastion
ssh -i bastion-key.pem ec2-user@<bastion-public-ip>

# Step 2 — From bastion, hop into private instance
ssh -i bastion-key.pem ec2-user@<private-instance-private-ip>

🗺️ CIDR Plan
ResourceCIDRIPs AvailableVPC (prod-vpc)10.0.0.0/1665,536Public subnet10.0.1.0/24256Private subnet10.0.2.0/24256DB subnet10.0.3.0/24256My IP (SSH rule)x.x.x.x/321 (just me)

✅ Key Learnings

Difference between stateful (SG) and stateless (NACL) firewalls — and why you need both
Why ephemeral ports 1024–65535 must be explicitly allowed in NACL outbound rules
Using Security Group ID as source instead of IP — production-grade pattern
Why DB subnet has no route table internet entry — no route = no path = fully isolated
How VPC Flow Logs capture metadata (not payload) for every network connection
Bastion host pattern — single controlled SSH entry point, private servers have zero public exposure


