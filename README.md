# Scalable Web App with Auto Scaling, ALB, and Custom VPC (AWS Project)

This project walks you through creating a fully operational and scalable AWS environment with:
- A custom VPC, subnets, and internet gateway
- EC2 instances in an Auto Scaling Group (ASG)
- An Application Load Balancer (ALB)
- Auto Scaling based on CPU usage
- CloudWatch-based scale-out demonstration

## Infrastructure Architecture

<img width="450" alt="1 - Infrastructure diagram" src="https://github.com/user-attachments/assets/ed69d390-9f62-4a12-b0c4-bb545ac04576" />


---

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

  
<img width="500" alt="6 - ALB Sec Group" src="https://github.com/user-attachments/assets/5bf0e106-7709-4d83-bdaa-63c0ce8d6e82" />


### Step 2: Create a VPC
- CIDR block: `10.0.0.0/16`
- Create and attach an **Internet Gateway (IGW)**

<img width="475" alt="2 - IGW create" src="https://github.com/user-attachments/assets/48c06010-e9b2-4146-8578-741dc48acb81" /><img width="475" alt="1 - VPC Create" src="https://github.com/user-attachments/assets/adfa5f78-3d2d-41b1-ab94-7f47e85d3f9b" />



### Step 3: Create Public Subnets
- Subnet 1: `10.0.1.0/24` in AZ1 (e.g., us-west-2a)
- Subnet 2: `10.0.2.0/24` in AZ2 (e.g., us-west-2b)

<img width="500" alt="3 - Subnets Created" src="https://github.com/user-attachments/assets/1f3fd24a-fa67-4f25-a106-99ffcfc1e6eb" />


### Step 4: Create a Public Route Table
- Route: `0.0.0.0/0` → IGW
- Associate the route table with both subnets

<img width="500" alt="4 - Route Table" src="https://github.com/user-attachments/assets/8d1b33e7-e422-4d76-bf08-571258e972f4" />


### Step 5: Create a Target Group
- Target Type: **Instance**
- Protocol: **HTTP (80)**
- Health Check: **Path `/`**, protocol HTTP
- Attach to the created VPC


<img width="500" alt="5 - Target Group" src="https://github.com/user-attachments/assets/c849e149-ed8f-49ac-a83d-5f5e5bb7162d" />


### Step 6: Create an Application Load Balancer (ALB)
- Internet-facing
- Listeners: HTTP (80)
- Attach to both subnets and the **ALB security group**
- Use the target group created earlier



<img width="500" alt="7 - ALB created" src="https://github.com/user-attachments/assets/6c7f0a95-7c51-44f8-9e74-9a1ad5b1a966" />


### Step 7: Create Auto Scaling Group
1. Go to **EC2 → Auto Scaling Groups → Create**.
2. Create a **Launch Template** with:
   - AMI: Amazon Linux 2
   - Instance Type: `t2.micro`
   - Security Group: **EC2 Security Group**
   - Enable Auto-assign Public IP
   - **User Data Script:** First script was used to create an AMI and then second script was used when creating the Lunch Template from the AMI
  
   
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

```
```bash
#!/bin/bash
IPADD=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
SECGR=$(curl -s http://169.254.169.254/latest/meta-data/security-groups)
INSTANCE=$(curl -s http://169.254.169.254/latest/meta-data/instance-type)
AZONE=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
echo '<h1>Public IP Address: IPADD </br>Security Group(s): SECGR </br>Availability Zone: AZONE </br>Instance Type: INSTANCE</h1>' > /var/www/html/index.txt
sed "s/AZONE/$AZONE/; s/IPADD/$IPADD/; s/SECGR/$SECGR/; s/INSTANCE/$INSTANCE/" /var/www/html/index.txt > /var/www/html/index.html
```


3. Configure Auto Scaling Group with:
   - VPC, subnets, and AZs created earlier
   - Attach the ALB and target group
   - Health check: **ELB**, grace period 20s
   - Desired capacity: 2
   - Min: 1
   - Max: 4

<img width="475" alt="8 - Lunch Template Created" src="https://github.com/user-attachments/assets/9c83bff0-225c-4356-a810-587c2bb6a506" /><img width="475" alt="9 - ASG created" src="https://github.com/user-attachments/assets/d8b180a7-862e-40b8-b783-d73e11a10f4d" />


### Step 8: Test the Setup
- Copy ALB DNS name from EC2 → Load Balancers.
- Visit the DNS in your browser → Refresh → Different instance metadata displayed with every refresh.

**Web App Server 1**

---
<img width="475" alt="10 - Load balancing Test 1a" src="https://github.com/user-attachments/assets/4b1d979a-f4d6-44f9-9d72-1d99fceda5d9" /> <img width="475" alt="10 - Load balancing Test 1b" src="https://github.com/user-attachments/assets/af2f594c-a027-47d3-9082-3079335050ce" />

---

**Web App Server 2**

---
<img width="475" alt="10 - Load balancing Test 2a" src="https://github.com/user-attachments/assets/c07b7937-8893-49cb-8bb5-6e0ac3c034a5" /> <img width="475" alt="10 - Load balancing Test 2b" src="https://github.com/user-attachments/assets/02656694-c021-47b3-b78c-4bd86d08832a" />

---


### Step 9: Enable Target Tracking Policy
- From Auto Scaling Group → Choose Automatic Scaling tab → Create Dynamic scaling policy
- Target Tracking Policy → Metric Type: **CPU Utilization**
- Target Value: **30%**
- Leave "Disable scale in to create only a scale-out policy" **Unchecked**
- This will auto create two alarms and link them to scaling policies in ASG:
   - **Alarm 1 (Scale Out)**: CPU > 30%
   - **Alarm 2 (Scale In)**: CPU < 27%

<img width="475" alt="11 - Target Tracking" src="https://github.com/user-attachments/assets/a5c1b250-1aa1-4db4-995e-a37e585cb91d" />


### Step 10: Stress Test an EC2 Instance
- Connect via SSH to one EC2 instance and run the below commands:
```bash
sudo yum install stress -y
sudo stress --cpu 12 --timeout 300s
```

<img width="475" alt="12 - Stress" src="https://github.com/user-attachments/assets/c169a235-957e-4013-9946-dd1df92b211e" />


### Step 12: Observe Scaling
- Monitor ASG → Maontoring → EC2 tab → CPU Utilization (Percent) and observe the spike.
- Monitor CloudWatch Alarms to see if the AarmHigh status is In Alarm.
- Monitor EC2 console to see if a new instance is launched automatically.
- Refresh your ALB DNS tab and observe the requested routed to the new instances launched.


 <img width="475" alt="13 - CPU Util Spike" src="https://github.com/user-attachments/assets/b82d3691-1ecc-41a8-a7f7-23be066ce323" /> <img width="475" alt="14 - In Alarm" src="https://github.com/user-attachments/assets/20a360fc-8bdc-4c92-8c7a-6749068cb51d" />
 


---

## ✅ Outcome
- Publicly accessible and load-balanced EC2 infrastructure
- Resilient to failure with self-healing and dynamic scaling
- Custom VPC for controlled networking

---


## 🧹 Recommended Deletion Order

To safely tear down this environment, delete AWS resources in the following order to avoid dependency errors:

1. **Auto Scaling Group (ASG)**  
   - Go to EC2 → Auto Scaling Groups → Delete

2. **Application Load Balancer (ALB)**  
   - Go to EC2 → Load Balancers → Delete

3. **Target Group (TG)**  
   - Go to EC2 → Target Groups → Delete

4. **Security Groups**  
   - Delete ALB and EC2-related security groups after detaching from resources

5. **Internet Gateway (IGW)**  
   - Detach from VPC → Delete

6. **VPC**  
   - After all associated resources (subnets, route tables, etc.) are deleted → Delete VPC

This ensures a clean and error-free teardown of your infrastructure.


## ✍️ Author

**Muhammad Nazir Said**  
