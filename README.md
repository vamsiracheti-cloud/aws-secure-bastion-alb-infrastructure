# Secure, Highly Available Multi-AZ Web Infrastructure on AWS

A production-ready, fault-tolerant infrastructure deployment on AWS featuring an isolated private network, secure administrative access, automated traffic routing, and self-healing scaling capabilities.

---

## 🗺️ System Architecture

* **Public Subnet (DMZ)**: Hosts a secure **Bastion Host (Jump Box)** acting as a single, hardened entry point into the VPC network.
* **Private Subnets (Isolated App Tier)**: Houses twin application instances distributed across distinct **Availability Zones (AZs)** with zero public internet exposure.
* **Application Load Balancer (ALB)**: Exposes a single public DNS endpoint to distribute client traffic across target zones.
* **Auto Scaling Group (ASG)**: Maintains a high-availability state across both availability zones, enforcing a self-healing fleet capacity.

---

## 🛠️ Step-by-Step Implementation & Engineering Milestones

### 1. Hardened Bastion Access & Multi-Hop SSH
* Configured the Bastion Security Group to strictly accept inbound SSH (Port 22) only from a designated administrative IP (`My IP`), preventing brute-force external attacks.
* Secured local credential files (`demo_vpc.pem`) with restrictive Linux file permissions to comply with SSH engine constraints:
  ```bash
  chmod 400 private_key.pem
  ```
* Bypassed network agent forwarding failures by setting up an **SSH Gateway (Jump Host) proxy configuration** in MobaXterm. This allowed multi-hop SSH routing straight to the isolated backend.

### 2. Multi-AZ Application Deployment & Cross-Zone Routing
* Implemented Python 3 web servers on private instances serving custom landing markers to test structural balancing.
* **Problem**: Initial terminal requests from the Bastion host repeatedly locked onto a single target instance due to local network affinity.
* **Resolution**: Enabled **Cross-Zone Load Balancing** at the AWS Target Group layer. This forced the ALB nodes to split traffic sequentially across distinct data centers, validating true round-robin routing via `curl` loop validation.

### 3. Transition to Production-Ready Services
* Migrated standard `nohup` manual terminal attachments into background **Linux `systemd` daemons** (`myapp.service`) on Ubuntu to enforce continuous uptime and automatic restart capabilities upon unexpected application crashes.

### 4. Self-Healing Fleet Automation
* Captured a specialized **Amazon Machine Image (AMI)** blueprint of the pre-configured environment.
* Configured an **AWS Auto Scaling Group (ASG)** backed by an independent Launch Template. 
* Validated full automated resiliency by executing simulated infrastructure failures (manually terminating healthy nodes), where the ASG successfully triggered automated recovery replacements to maintain desired state capacity.
