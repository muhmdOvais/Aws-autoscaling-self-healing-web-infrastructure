# AWS Auto Scaling and Self-Healing Web Infrastructure

## Project Architecture

User → Application Load Balancer → Auto Scaling Group → EC2 Instances → CloudWatch → SNS

📷 Add Screenshot Here:

* Architecture Diagram

---

# 🚀 Phase 1 — Web Server Deployment and AMI Creation

## Step 1 Launch EC2 Instance

* Launch Ubuntu t2.micro instance
* Configure Security Group:

  * SSH (22)
  * HTTP (80)
* Download key pair

---

## Step 2 Connect to EC2

```bash
ssh -i aws-devops-web-key.pem ubuntu@<public-ip>
```

---

## Step 3 Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl status nginx
```

Expected:

```bash
active (running)
```

📷 Add Screenshot Here:

* Nginx Running Status

---

## Step 4 Deploy Static Website to EC2

Upload files from local system:

```bash
scp -r -i ~/aws-devops-web-key.pem /mnt/d/DevOps/projects/project1/* ubuntu@<public-ip>:/home/ubuntu/
```

Copy into Nginx web root:

```bash
sudo cp -r /home/ubuntu/* /var/www/html/
sudo chmod -R 755 /var/www/html
sudo systemctl restart nginx
```

Verify:

```bash
ls /var/www/html
```

Expected:

```bash
index.html
images/
tooplate-graphite-creative.css
tooplate-graphite-script.js
```

Access:

```text
http://<public-ip>
```

📷 Add Screenshot Here:

* Website Running on EC2

---

## Step 5 Create AMI

EC2 → Actions → Image and Templates → Create Image

Name:

```text
aws-devops-web-ami
```

Wait until status:

```text
Available
```

---

# 🚀 Phase 2 — Scaling Infrastructure

## Step 6 Create Launch Template

Use:

* Custom AMI
* t2.micro
* Existing Security Group
* Existing Key Pair

Name:

```text
aws-devops-launch-template
```

---

## Step 7 Create Auto Scaling Group

Configuration:

```text
Minimum = 1
Desired = 2
Maximum = 5
```

Select two subnets for high availability.

📷 Add Screenshot Here:

* Auto Scaling Group Configuration

---

## Step 8 Attach Application Load Balancer

Choose:

```text
Attach to a new load balancer
Application Load Balancer
Internet Facing
HTTP 80
```

---

## Step 9 Verify Target Group Health

Expected:

```text
2 Healthy Targets
```

📷 Add Screenshot Here:

* Target Group Healthy Targets

---

## Step 10 Self-Healing Test

Terminate one ASG instance manually.

Observe:

* Instance terminated
* Replacement instance auto-created
* Target group returns healthy

📷 Add Screenshot Here:

* Before Termination
* After Replacement Instance

---

# 🚀 Phase 3 — Monitoring and Alerts

## Step 11 Create SNS Topic

Create topic:

```text
cpu-alert-topic
```

Create email subscription and confirm email.

---

## Step 12 Create CloudWatch Alarm

Metric:

```text
CPUUtilization
```

Condition:

```text
Less than 10%
```

Evaluation:

```text
1 out of 1 datapoints within 5 minutes
```

Action:

```text
Send notification to cpu-alert-topic
```

Alarm Name:

```text
low-cpu-alert
```

---

## Step 13 Verify Alarm Trigger

Expected State:

```text
In Alarm
```

📷 Add Screenshot Here:

* CloudWatch Alarm Triggered

---

# 🧠 How the Architecture Works

## EC2

Hosts the website using Nginx.

## AMI

Creates reusable server snapshot.

## Launch Template

Defines how new instances are launched.

## Auto Scaling Group

Maintains desired instance count.

## Application Load Balancer

Distributes traffic to healthy instances.

## Target Group

Tracks backend instances and health checks.

## CloudWatch

Monitors metrics such as CPU utilization.

## SNS

Sends notification when alarms trigger.

---

# ⚠ Problems Faced and How They Were Solved

## Problem 1 — SSH Key Permission Error

Error:

```bash
WARNING: UNPROTECTED PRIVATE KEY FILE
```

Fix:

```bash
cp /mnt/c/Users/user/Downloads/aws-devops-web-key.pem ~/
chmod 400 ~/aws-devops-web-key.pem
```

---

## Problem 2 — Load Balancer Not Working

Issue:

```text
0 Healthy Targets
0 Total Targets
```

Cause:
ASG instances were not attached to target group.

Fix:

* Edit Auto Scaling Group
* Attach existing target group
* Wait for targets to register

Result:

```text
2 Healthy Targets
```

📷 Add Screenshot Here:

* 0 Targets Issue
* 2 Healthy Fixed

---

## Problem 3 — Security Group Missing HTTP Rule

Fix:
Added:

```text
HTTP 80 → 0.0.0.0/0
```

This allowed ALB health checks.

---

## Problem 4 — Self Healing Delay Confusion

Observation:
Replacement instance took time.

Explanation:
AWS needed to:

* Detect termination
* Launch replacement
* Boot from AMI
* Pass health checks
* Register in target group

Normal behavior.

---

# 💻 Commands Used in Project

```bash
ssh -i key.pem ubuntu@public-ip
sudo apt update
sudo apt install nginx -y
sudo systemctl status nginx
scp -r -i key.pem project/* ubuntu@public-ip:/home/ubuntu/
sudo cp -r /home/ubuntu/* /var/www/html/
sudo chmod -R 755 /var/www/html
sudo systemctl restart nginx
ls /var/www/html
```

---

# 📸 Suggested Screenshots for GitHub

Add these:

1 Architecture Diagram
2 Nginx Running Status
3 Website Live on EC2/ALB
4 Auto Scaling Group Configuration
5 Target Group Healthy Targets
6 Self-Healing Replacement Instance
7 CloudWatch Alarm In Alarm
8 Issue Screenshot (0 targets) and Fix Screenshot (2 healthy)

---

# 🎯 Key Concepts Demonstrated

* Static Web Hosting on EC2
* Auto Scaling
* Self Healing Infrastructure
* Load Balancing
* Target Group Health Checks
* Monitoring and Alerting
* Troubleshooting AWS Networking

---

# 🧹 Cleanup (Avoid Charges)

Delete in this order:

## 1 Delete Auto Scaling Group

Set:

```text
Desired 0
Min 0
Max 0
```

Then delete ASG.

---

## 2 Terminate EC2 Instances

Delete:

* Original EC2
* ASG instances

---

## 3 Delete Load Balancer

Important:
ALBs can incur charges.

---

## 4 Delete Target Group

---

## 5 Delete Launch Template

---

## 6 Deregister AMI

---

## 7 Delete EBS Snapshot

Important.
Snapshots can cost money.

---

## 8 Delete CloudWatch Alarm

---

## 9 Delete SNS Topic

---

# ⚠ Cost Traps to Avoid

Resources that may cost money if left running:

* Running EC2 instances
* Stopped EC2 EBS volumes
* Application Load Balancer
* AMI snapshots

Always terminate and delete after practice.

---

# 💡 Conclusion

This project demonstrates deployment of a scalable, fault-tolerant and monitored web infrastructure using core AWS services and includes real-world troubleshooting scenarios encountered and resolved during implementation.
