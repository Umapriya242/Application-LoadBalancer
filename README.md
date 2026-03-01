# AWS Application Load Balancer Project

This project demonstrates advanced traffic distribution using Amazon Web Services (AWS) Application Load Balancer (ALB) with multiple EC2 instances, Target Groups, Auto Scaling concepts, and monitoring using Amazon CloudWatch.

---

## 1. Launch EC2 Instances

### Instance Configuration

- Created 3 EC2 instances: `m1`, `m2`, `m3`
- AMI: Amazon Linux 2
- Instance Type: t2.micro
- Key Pair: Configured for SSH access

### Security Group Rules

- HTTP (Port 80) → 0.0.0.0/0 (Allow public web access)
- SSH (Port 22) → Restricted to your IP (Secure remote access)

This ensures public users can access the website while administrative access remains restricted.

---

## 2. User Data Configuration

Used EC2 User Data scripts for automatic provisioning:

- Installed Apache Web Server
- Started and enabled `httpd` service
- Created folders inside `/var/www/html/`:
  - `/Site1/index.html`
  - `/Site2/index.html`
- Added unique content in each instance to identify which server handled the request

This automated configuration ensures consistency across instances.

---

## 3. Create Target Groups

Created 4 Target Groups:

| Target Group | Registered Targets |
|--------------|-------------------|
| t1 | m1 |
| t2 | m2 |
| t3 | m3 |
| t4 | m1, m2, m3 |

### Target Group Configuration

- Target Type: Instance
- Protocol: HTTP
- Port: 80
- Health Check Path: `/folder1/index.html`
- Health Check Interval: 30 seconds
- Healthy Threshold: 5
- Unhealthy Threshold: 2

The health check ensures only responsive instances receive traffic.

---

## 4. Create Application Load Balancer (ALB)

Configured an Internet-Facing Application Load Balancer:

- Listener: HTTP (Port 80)
- Scheme: Internet-facing
- IP Address Type: IPv4
- Availability Zones: Enabled in 2+ AZs for high availability
- Security Group: Allow HTTP (Port 80)

Attached the created Target Groups to the ALB.

The ALB acts as the entry point for all incoming traffic.

---

## 5. Configure Listener Rules (Path-Based Routing)

### Default Rule

- Forward to `t4` (Load balanced across m1, m2, m3)

### Custom Path-Based Rules

- `/Site1/*` → Forward to `t1` (Only m1)
- `/Site3/*` → Forward to `t2` (Only m2)

### Priority Settings

- Lower number = Higher priority
- Path-based rules placed above default rule

This configuration enables traffic routing based on URL paths.

---

## 6. Testing the Setup

Accessed the ALB DNS name:

- `/site1/index.html` → Routed to m1
- `/site3/index.html` → Routed to m2
- `/` → Distributed across m1, m2, m3 (via t4)

### Fault Simulation

- Manually stopped one EC2 instance
- Observed:
  - Health check marked instance as unhealthy
  - ALB automatically stopped sending traffic to it
  - Remaining healthy instances handled traffic

This validated automatic failover capability.

---

## 7. Monitoring

Used Amazon CloudWatch to monitor:

- EC2 CPU Utilization
- Network In/Out
- ALB Request Count
- Healthy Host Count
- Target Response Time

Monitoring ensured real-time visibility into:

- Instance health
- Traffic distribution
- Scaling activity

---

## Project Summary

This project demonstrates advanced traffic distribution and scalability using AWS Application Load Balancer with:

- 3 EC2 instances
- 4 Target Groups
- Path-based routing
- Health monitoring and automatic failover
