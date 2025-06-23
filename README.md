# Scalable Web App with Auto Scaling, ALB, and Custom VPC (AWS Project)

This project walks you through creating a fully operational and scalable AWS environment with:
- A custom VPC, subnets, and internet gateway
- EC2 instances in an Auto Scaling Group (ASG)
- An Application Load Balancer (ALB)
- Auto Scaling based on CPU usage
- CloudWatch-based scale-out demonstration

## Prerequisites
- An AWS account
- IAM permissions to manage EC2, VPC, ALB, ASG, and CloudWatch

---

## 🛠️ Step-by-Step Deployment Guide

### Step 1: Create Security Groups
- **ALB Security Group**: Allow inbound HTTP (80) from `0.0.0.0/0`
- **EC2/ASG Security Group**:
  - Allow HTTP (80) **from ALB security group**
  - Allow SSH (22) **from your IP or anywhere (`0.0.0.0/0`)**

### Step 2: Create a VPC
- CIDR block: `10.0.0.0/16`
- Create and attach an **Internet Gateway (IGW)**

### Step 3: Create Public Subnets
- Subnet 1: `10.0.1.0/24` in AZ1 (e.g., us-east-1a)
- Subnet 2: `10.0.2.0/24` in AZ2 (e.g., us-east-1b)

### Step 4: Create a Public Route Table
- Route: `0.0.0.0/0` → IGW
- Associate the route table with both subnets

### Step 5: Create a Target Group
- Target Type: **Instance**
- Protocol: **HTTP (80)**
- Health Check: **Path `/`**, protocol HTTP
- Attach to the created VPC

### Step 6: Create an Application Load Balancer (ALB)
- Internet-facing
- Listeners: HTTP (80)
- Attach to both subnets and the **ALB security group**
- Use the target group created earlier

### Step 7: Create Auto Scaling Group
1. Go to **EC2 → Auto Scaling Groups → Create**.
2. Create a **Launch Template** with:
   - AMI: Amazon Linux 2
   - Instance Type: `t2.micro`
   - Security Group: **EC2 Security Group**
   - Enable Auto-assign Public IP
   - **User Data Script:**

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
IPADD=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
SECGR=$(curl -s http://169.254.169.254/latest/meta-data/security-groups)
INSTANCE=$(curl -s http://169.254.169.254/latest/meta-data/instance-type)
AZONE=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
echo "<h1>Public IP Address: $IPADD <br>Security Group(s): $SECGR <br>Availability Zone: $AZONE <br>Instance Type: $INSTANCE</h1>" > /var/www/html/index.html
```

3. Configure Auto Scaling Group with:
   - VPC, subnets, and AZs created earlier
   - Attach the ALB and target group
   - Health check: **ELB**, grace period 20s
   - Desired capacity: 2
   - Min: 1
   - Max: 4

### Step 8: Test the Setup
- Copy ALB DNS name from EC2 → Load Balancers.
- Visit in browser → Refresh → Different instance metadata displayed.

### Step 9: Enable Target Tracking Policy
- Metric Type: **CPU Utilization**
- Target Value: **30%**
- Scaling policy: Auto scale in and out

### Step 10: Manual Auto Scaling with CloudWatch
1. Create two alarms:
   - **Alarm 1 (Scale Out)**: CPU > 60% for 2 consecutive periods
   - **Alarm 2 (Scale In)**: CPU < 20% for 2 consecutive periods
2. Link alarms to scaling policies in ASG

### Step 11: Stress Test an EC2 Instance
- Connect via SSH to one EC2 instance:
```bash
sudo yum install stress -y
sudo stress --cpu 12 --timeout 300s
```

### Step 12: Observe Scaling
- Monitor ASG and EC2 console to see if a new instance is launched automatically.

---

## ✅ Outcome
- Publicly accessible and load-balanced EC2 infrastructure
- Resilient to failure with self-healing and dynamic scaling
- Custom VPC for controlled networking

---

## 📁 Files Provided
- `README.md`: Full walkthrough
---

## 🧹 Recommended Deletion Order

To safely tear down this environment, delete AWS resources in the following order to avoid dependency errors:

1. **Auto Scaling Group (ASG)**  
   - Go to EC2 → Auto Scaling Groups → Delete

2. **Target Group (TG)**  
   - Go to EC2 → Target Groups → Delete

3. **Application Load Balancer (ALB)**  
   - Go to EC2 → Load Balancers → Delete

4. **Security Groups**  
   - Delete ALB and EC2-related security groups after detaching from resources

5. **Internet Gateway (IGW)**  
   - Detach from VPC → Delete

6. **VPC**  
   - After all associated resources (subnets, route tables, etc.) are deleted → Delete VPC

This ensures a clean and error-free teardown of your infrastructure.
