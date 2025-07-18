# 🚀 vProfile AWS Lift-and-Shift Setup Instructions

This guide covers the deployment of the **vProfile** Java-based application to AWS using a Lift-and-Shift strategy. It includes EC2 setup, security configurations, DNS setup, artifact deployment, load balancing, and auto scaling.

---

## 🔐 Security & Key Pair Configuration

### 1. Security Group: `vprofile-ELB-sg` (Load Balancer)
- **Purpose**: Allows public internet traffic to reach the Application Load Balancer.
- **Inbound Rules**:
  - `HTTP (80)` from Anywhere (IPv4 & IPv6)
  - `HTTPS (443)` from Anywhere (IPv4 & IPv6)
- ⚠️ Do not modify outbound rules.

### 2. Security Group: `vprofile-app-sg` (Tomcat Instances)
- **Purpose**: Allows traffic from Load Balancer and SSH access.
- **Inbound Rules**:
  - `TCP 8080` from `vprofile-ELB-sg`
  - `SSH 22` from **Your IP**
- 💡 Update the SSH rule when your IP changes.

### 3. Security Group: `vprofile-backend-sg` (MySQL, Memcache, RabbitMQ)
- **Purpose**: Controls backend service access.
- **Inbound Rules**:
  - `TCP 3306` from `vprofile-app-sg` (MySQL)
  - `TCP 11211` from `vprofile-app-sg` (Memcached)
  - `TCP 5672` from `vprofile-app-sg` (RabbitMQ)
  - `SSH 22` from **Your IP**
  - `ALL Traffic` from `vprofile-backend-sg` for internal communication

### 🔑 Key Pair: `vprofile-prod-key`
- PEM format key for SSH access to EC2
- **Keep this file secure and private**

---

## 🖥️ EC2 Instance Setup and Verification

### EC2 Instances and Scripts

| Instance Name    | Purpose    | AMI Used         | Script Used         | Security Group         |
|------------------|------------|------------------|----------------------|--------------------------|
| vprofile-db01    | MySQL DB   | Amazon Linux 2023| `mysql.sh`           | `vprofile-backend-sg`    |
| vprofile-mc01    | Memcached  | Amazon Linux 2023| `memcache.sh`        | `vprofile-backend-sg`    |
| vprofile-rmq01   | RabbitMQ   | Amazon Linux 2023| `rabbitmq.sh`        | `vprofile-backend-sg`    |
| vprofile-app01   | Tomcat     | Ubuntu 24.04     | `tomcat_ubuntu.sh`   | `vprofile-app-sg`        |

### Setup Summary
- Clone repo: `https://github.com/hkhcoder/vprofile-project`
- Checkout branch: `aws-liftandshift`
- Tags: Name (e.g., vprofile-db01), Project: vprofile

### Security Group Mapping
- **App SG** → Tomcat (8080), SSH
- **Backend SG** → DB (3306), Memcached (11211), RabbitMQ (5672), SSH
- **LB SG** → HTTP/HTTPS

### Post-Launch Verification
- Check services via `ssh` + `systemctl status`
- Verify DB tables, RabbitMQ queue, Memcached status

---

## 🌐 Route 53 Private DNS

### Why?
- Prevent hardcoded IPs in app
- Dynamic name resolution for backend services

### Example DNS Records

| Record Name | Type | Value (Private IP)         | Service        |
|-------------|------|-----------------------------|----------------|
| db01        | A    | vprofile-db01 private IP    | MySQL          |
| mc01        | A    | vprofile-mc01 private IP    | Memcache       |
| rmq01       | A    | vprofile-rmq01 private IP   | RabbitMQ       |

### Verify DNS (from App Instance)
```bash
ping db01.vprofile.internal
ping mc01.vprofile.internal
ping rmq01.vprofile.internal
```

---

## ☁️ Java WAR Deployment via Maven, S3, and EC2

### Tools Needed
- **Maven**, **JDK 17**, **AWS CLI**, **S3 Bucket**
- **IAM User** (for local), **IAM Role** (for EC2)

### Local Computer Steps
1. Install Maven, JDK, AWS CLI
2. Configure AWS CLI: `aws configure`
3. Update `application.properties` with DNS names
4. Build: `mvn install`
5. Upload: `aws s3 cp target/vprofile-v2.war s3://vprofile-las-artifacts69`

### EC2 Server Steps
1. SSH into EC2
2. Install AWS CLI
3. Download WAR from S3
4. Deploy to Tomcat

---

## ⚖️ Load Balancer Setup

### Target Group
- Name: `vprofile-las-tg`
- Port: 8080 (override in health check)
- Register app01

### ALB Configuration
- Name: `vprofile-las-elb`
- Ports: 80/443
- Target: `vprofile-las-tg`
- Optional: Add HTTPS with ACM Certificate

### Validate
- Open browser: `http://<LB-DNS>` or `https://<domain>`

---

## 🔄 Auto Scaling Configuration

### Steps:
1. Create AMI from app01
2. Create Launch Template using AMI
3. Create Auto Scaling Group
   - Min: 1, Max: 4, Target: CPU 50%
   - Attach to ALB Target Group

---

## 🧠 Lift-and-Shift Scope
- No data migration (just code & infra)
- Used IaaS model: all services installed on EC2

---

## ✅ Final Architecture Summary

- **Security**: SGs, Key Pairs, IAM
- **Compute**: EC2 + Auto Scaling Group
- **Storage**: S3 for artifact
- **Deploy**: Maven → S3 → EC2 Tomcat
- **Routing**: ALB + Route 53 (Private DNS)
- **Scaling**: ASG with CPU metrics
