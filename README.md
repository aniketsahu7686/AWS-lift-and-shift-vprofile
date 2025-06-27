# AWS-lift-and-shift-vprofile


----------------------------------------------------------------- 📌Project Overview ----------------------------------------------------------------------
This is a cloud migration project where we implement a Lift and Shift strategy to migrate a multi-tier web application — vProfile — from a virtualized local environment to the AWS Cloud.

The goal is to simulate how real-world legacy workloads are moved from physical or virtual data centers to the cloud without changing the core architecture or application code.



----------------------------------------------------------------- 🎯Objectives ----------------------------------------------------------------------
✅ Understand and implement the Lift and Shift cloud migration strategy

✅ Migrate vProfile, a Java-based web application, to AWS

✅ Leverage IaaS (Infrastructure as a Service) model of AWS

✅ Enable scalability, flexibility, and automation

✅ Use AWS services for load balancing, auto scaling, DNS, and storage

✅ Implement private DNS resolution, secure HTTPS access, and cost-efficient architecture

🧱 Original Architecture (Before Migration)
Previously hosted locally using Vagrant virtual machines, the application stack included:

🌐 Nginx – Reverse Proxy

🐱 Apache Tomcat – Application Server

🐬 MySQL – Database

🧠 Memcached – Caching Layer

📨 RabbitMQ – Messaging Queue

☁️ AWS Cloud Architecture (Post-Migration)
✅ AWS Services Used
AWS Service	Purpose
EC2	Hosts Tomcat, MySQL, Memcached, and RabbitMQ
Amazon S3	Stores application artifacts (WAR files)
Elastic Load Balancer (ALB)	Distributes traffic to Tomcat instances
Auto Scaling Group	Scales Tomcat instances based on load
Amazon Route 53	Provides private DNS for backend services
AWS Certificate Manager (ACM)	Enables HTTPS with SSL certificates
IAM	Manages user access and roles
EBS	Persistent storage for each EC2 instance
Key Pairs	Secure access to EC2 instances

📡 Architecture Flow
🌍 User Access:
Users access the application using a custom domain configured in GoDaddy DNS.

🔒 HTTPS Routing:
Traffic is routed to an Application Load Balancer (ALB) secured by an SSL certificate from ACM.

⚖️ Load Balancing:
The ALB forwards traffic to Tomcat EC2 instances via port 8080.

📈 Auto Scaling:
Tomcat instances are managed by an Auto Scaling Group for dynamic scalability.

🔗 Backend Communication:
Tomcat instances interact with MySQL, RabbitMQ, and Memcached via private DNS using Route 53.

🧭 Private DNS Resolution:
Private IPs of backend services are mapped to DNS entries like mysql.vprofile.internal.

📦 Artifact Deployment
🛠️ Application is built locally

☁️ WAR file is uploaded to Amazon S3

📥 Tomcat EC2 instances download and deploy the WAR file from S3 automatically

🔐 Security Architecture
Each component is placed in a dedicated security group

Only least-privilege access rules are applied

Example:

Load balancer allows only HTTPS (443) inbound

Tomcat instances allow only port 8080 from ALB

Backend services allow access only from Tomcat instances

