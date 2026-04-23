# AWS Auto Scaling and Self-Healing Web Infrastructure

## Project Architecture

User → Application Load Balancer → Auto Scaling Group → EC2 Instances → CloudWatch → SNS

* Architecture Diagram
<img width="1536" height="1024" alt="4060f7ef-228d-497f-9429-23d7ec6c8c2f" src="https://github.com/user-attachments/assets/f7999222-b3e3-4419-8db7-6768a3e00537" />

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

* Nginx Running Status
<img width="1919" height="913" alt="Screenshot 2026-04-23 004701" src="https://github.com/user-attachments/assets/f9bb6dda-c70d-49bd-9046-a192d7d69db0" />

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
* Website Running on EC2
<img width="1919" height="955" alt="Screenshot 2026-04-23 005332" src="https://github.com/user-attachments/assets/631396a8-908c-4b24-ad8f-a2b522e60495" />

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

* Auto Scaling Group Configuration
<img width="1919" height="907" alt="Screenshot 2026-04-23 002602" src="https://github.com/user-attachments/assets/72f80ac5-f800-447a-87d7-dce258cbba89" />

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
* Target Group Healthy Targets
<img width="1919" height="910" alt="Screenshot 2026-04-23 002804" src="https://github.com/user-attachments/assets/0b50b342-c1d8-4e64-a0f0-e60649fcc416" />

---

## Step 10 Self-Healing Test

Terminate one ASG instance manually.

Observe:

* Instance terminated
* Replacement instance auto-created
* Target group returns healthy

<img width="1919" height="913" alt="Screenshot 2026-04-23 003204" src="https://github.com/user-attachments/assets/b353abc4-5214-49c8-9ce5-f1e6ed9ba496" /> 
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
* CloudWatch Alarm Triggered
<img width="1919" height="911" alt="Screenshot 2026-04-23 003342" src="https://github.com/user-attachments/assets/f0c69c3f-9504-4066-bd7d-1e995ed0ad2a" />

---

# 🧠 How the Architecture Works
<img width="1536" height="1024" alt="4060f7ef-228d-497f-9429-23d7ec6c8c2f" src="https://github.com/user-attachments/assets/9d35fa04-49bd-44c7-998f-bcc190434241" />


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
