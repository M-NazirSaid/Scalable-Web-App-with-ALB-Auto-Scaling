# Scalable Web App with ALB & Auto Scaling (AWS Project)

This project demonstrates how to deploy a highly available and scalable web application using **Amazon EC2**, **Application Load Balancer (ALB)**, and **Auto Scaling**.

## Architecture Overview
- EC2 instances serve the web app.
- ALB distributes traffic across instances.
- Auto Scaling ensures scalability and availability across multiple Availability Zones.

## Step-by-Step Deployment Guide

### Step 1: Create a Launch Template
1. Navigate to AWS Console → EC2 → Launch Templates → Create Launch Template.
2. Use the following configuration:
   - **Name:** `WebAppTemplate`
   - **AMI:** Amazon Linux 2 or Ubuntu
   - **Instance Type:** `t2.micro` or `t3.medium`
   - **Key Pair:** Select an existing or create a new one
   - **User Data Script:** See `user-data.sh` for automatic Apache setup
   - **Security Group:** Allow `SSH (22)`, `HTTP (80)`, and `HTTPS (443)`

### Step 2: Create an Auto Scaling Group
1. Go to EC2 → Auto Scaling Groups → Create Auto Scaling Group.
2. Choose the Launch Template created above.
3. Configure group size:
   - **Desired Capacity:** 2
   - **Min:** 1
   - **Max:** 4
4. Select a VPC and at least two subnets across different AZs.
5. Attach Application Load Balancer:
   - Create a **Target Group** (Type: Instance, Protocol: HTTP, Health Check: `/`)
   - Register instances later
6. Set scaling policies:
   - Scale out: CPU > 60%
   - Scale in: CPU < 40%

### Step 3: Create an Application Load Balancer (ALB)
1. Go to EC2 → Load Balancers → Create Load Balancer.
2. Select **Application Load Balancer** and configure:
   - **Name:** `WebAppALB`
   - **Scheme:** Internet-facing
   - **Listeners:** HTTP on port 80
   - **Target Group:** Use the one created earlier
   - **Security Group:** Allow `HTTP (80)`

### Step 4: Test the Setup
- Go to EC2 → Load Balancers → Copy the ALB DNS Name.
- Visit `http://<ALB-DNS-Name>` in a browser.
- You should see: **Welcome to Scalable Web App**.

### Step 5: Verify Auto Scaling
- Increase traffic using Apache Benchmark:
  ```bash
  ab -n 1000 -c 50 http://<ALB-DNS-Name>/
  ```
- Monitor Auto Scaling Group to verify if new instances are launched.
- Stop an instance to see if it gets replaced automatically.

## Conclusion
✅ High Availability using ALB  
✅ Scalability using Auto Scaling  
✅ Redundancy across multiple Availability Zones

---

**Files Included**
- `user-data.sh`: Initialization script for EC2 instances
- `README.md`: Project walkthrough
