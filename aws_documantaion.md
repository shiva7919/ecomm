# 🚀 Deploying an E-Commerce App on AWS EC2 with Load Balancer and Auto Scaling

This guide provides a step-by-step process to deploy a scalable, fault-tolerant E-Commerce application on AWS using EC2 instances, an Application Load Balancer, and Auto Scaling.

# 📘 Project Description

This project involves the end-to-end deployment of a robust and production-ready **E-Commerce web application** on **Amazon Web Services (AWS)**. It showcases how to design and implement a **fault-tolerant**, **highly available**, and **scalable architecture** using key AWS services such as **EC2**, **Application Load Balancer (ALB)**, and **Auto Scaling Groups (ASG)**.

The main goal of this deployment is to ensure:

- **High Availability:** The application is distributed across multiple Availability Zones to reduce downtime and handle zone-level failures gracefully.
- **Scalability:** Auto Scaling policies are configured to dynamically increase or decrease the number of EC2 instances based on real-time CPU utilization, ensuring efficient resource usage and cost optimization.
- **Load Balancing:** An Application Load Balancer distributes incoming HTTP requests across healthy EC2 instances, enhancing performance and resilience.
- **Infrastructure as a Service:** The infrastructure is provisioned using AWS native services, offering flexibility and control over compute resources.
- **Monitoring and Alerts:** Integration with Amazon CloudWatch enables real-time monitoring and auto-triggered scaling actions. Optional integration with Amazon SNS can be used to send email notifications about system events.

This project simulates a real-world use case of hosting a customer-facing web application that is expected to handle varying levels of traffic without any manual intervention. The application is served through **Apache Web Server** on Amazon Linux, and all configurations are designed to be modular and reusable for other production workloads.

By following this deployment model, businesses can ensure that their application is always available, responsive, and cost-efficient — regardless of user demand.

---

## 🧱 Architecture Overview

### 🧩 Components Involved:

- Amazon EC2 (Elastic Compute Cloud)
- Security Groups
- Application Load Balancer (ALB)
- Target Groups
- Amazon Machine Image (AMI)
- Launch Template
- Auto Scaling Group
- Amazon CloudWatch
- Amazon SNS (Optional for notifications)

---

### 🔁 Architecture Flow:

```plaintext
User Request
     ↓
Application Load Balancer (ALB)
     ↓
Target Group
     ↓
Auto Scaling Group (EC2 Instances in multiple AZs)
     ↓
Apache Web Server serving E-Commerce App
```
---

## 1. 🏗️ Provision EC2 Infrastructure

### Step 1.1: Create a Security Group

- Go to: **EC2 > Security Groups**
- Click **Create Security Group**
- Provide the following:
  - **Name**: `Ecom-SG`
  - **Description**: `Security group for EC2 instances`

#### Add Inbound Rules:
| Type  | Protocol | Port Range | Source            |
|-------|----------|------------|-------------------|
| SSH   | TCP      | 22         | Anywhere (IPv4)   |
| HTTP  | TCP      | 80         | Anywhere (IPv4)   |

- Click **Create Security Group**
![Image](https://github.com/user-attachments/assets/45cb636d-0b6e-47bf-9b85-4a5cdffa79f1)
---

## 2. 🚚 Deploy the E-Commerce Application

### Step 2.1: Launch EC2 Instances

- Navigate to: **EC2 > Instances > Launch Instances**
- Create two instances:

| Name     | Availability Zone |
|----------|-------------------|
| EC2-1A   | us-east-1a        |
| EC2-1B   | us-east-1b        |

#### Configuration:
- **AMI**: Amazon Linux AMI
- **Instance Type**: t2.micro (or your choice)
- **Security Group**: `Ecom-SG`
- **Key Pair**: Create new named `joy`
- Leave all other settings default

![Image](https://github.com/user-attachments/assets/b87146fe-990c-4429-9597-0edc015f96a4)
![Image](https://github.com/user-attachments/assets/f71291fe-4cca-4290-80c7-1e99325f1d3f)
---

### Step 2.2: SSH into Each Instance and Deploy the App

Run the following commands on **both instances**:

```bash
sudo yum update -y
sudo yum install git httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo git clone https://github.com/shiva7919/ecomm.git
sudo rm -rf /var/www/html/*
sudo cp -r ecomm/* /var/www/html
```

### Step 2.3: Access the Application

Open your browser and access the application using the public IPs of both EC2 instances:

- [http://<public-ip-of-EC2-1A>:80](http://<public-ip-of-EC2-1A>:80)
- [http://<public-ip-of-EC2-1B>:80](http://<public-ip-of-EC2-1B>:80)

![Image](https://github.com/user-attachments/assets/a672c132-6bdb-4a99-a960-d3970bf5569e)

> 🔎 Replace `<public-ip-of-EC2-1A>` and `<public-ip-of-EC2-1B>` with the actual public IP addresses of your instances.
>
> ## 3. ⚖️ Configure Load Balancer

### Step 3.1: Create Target Group

1. Navigate to: **EC2 > Target Groups > Create Target Group**
2. Configure the following:
   - **Target Type**: Instances
   - **Name**: `Ecom-TG`
   - **Protocol**: HTTP
   - **Port**: 80
   - **VPC**: Default VPC
3. Register both EC2 instances to the target group.
4. Click **Create Target Group**
![Image](https://github.com/user-attachments/assets/effcfa91-66c1-489e-a090-d68ae8ecc7bb)
---

### Step 3.2: Create Application Load Balancer (ALB)

1. Navigate to: **EC2 > Load Balancers > Create Load Balancer**
2. Choose: **Application Load Balancer**

#### Configuration:
- **Name**: `Ecom-ALB`
- **Scheme**: Internet-facing
- **Listener**: HTTP on port 80
- **Availability Zones**: `us-east-1a`, `us-east-1b`
- **Security Group**: `Ecom-SG`
- **Target Group**: `Ecom-TG`

3. Click **Create Load Balancer**

---

### Step 3.3: Test Load Balancer

Once the ALB status becomes **Active**, open the following URL in your browser:

![Image](https://github.com/user-attachments/assets/1e12e475-6b6c-411b-8826-0b2d920c6826)
![Image](https://github.com/user-attachments/assets/7256d2ac-d2bd-4abb-aff6-db4333d28081)


> 🔎 Replace `<load-balancer-dns>` with the actual DNS name of your load balancer.
>![Image](https://github.com/user-attachments/assets/d2062228-50a3-4830-96f4-abeabc857542)
> You should see the E-Commerce application successfully load through the ALB.
## 4.  Set Up Auto Scaling

### Step 4.1: Create an AMI from Existing Instance

1. Go to **EC2 > Instances**.
2. Select one of the deployed instances.
3. Click **Actions > Image and templates > Create image**.
4. Name the image as `Ecom-AMI`.
5. Uncheck the **Reboot** option.
6. Click **Create image**.
7. Monitor the status under **EC2 > AMIs** until it shows **Available**.
![Image](https://github.com/user-attachments/assets/f29285fb-d268-49ce-bdd5-e9b5d40cc4c2)
![Image](https://github.com/user-attachments/assets/1c9cd065-639b-4aac-846a-7044ebfe9549)
![Image](https://github.com/user-attachments/assets/fad28300-1188-4b3d-8dd0-9277a10febcd)

### Step 4.2: Create a Launch Template

1. Navigate to **EC2 > Launch Templates > Create launch template**.

![Image](https://github.com/user-attachments/assets/18a7fc4f-1e2c-4f49-8b05-4864fda65cc2)

2. Set:

   * **Template name:** `Ecom-LTEMP`
   * **AMI:** Select from “Owned by me” → choose `Ecom-AMI`
   * **Instance type:** `t2.micro`
   * **Key Pair:** `joy`
   * **Security Group:** `Ecom-SG`
3. Click **Create launch template**.

![Image](https://github.com/user-attachments/assets/dd323e1c-1225-4285-986e-75d76d02d5e6)

![Image](https://github.com/user-attachments/assets/e70d3fb2-995a-43e6-b1e7-1862068afe42)

### Step 4.3: Create Auto Scaling Group

1. Go to **EC2 > Auto Scaling Groups > Create Auto Scaling group**.

![Image](https://github.com/user-attachments/assets/a1023d55-d837-4f72-b6e1-9ea0e1b4e835)

2. Name it: `Ecom-ASG`.
3. Select **Launch Template:** `Ecom-LTEMP`.
4. Select **Availability Zones:** `us-east-1a` and `us-east-1b`.
5. Attach the existing load balancer:

   * Choose the **Application Load Balancer**
   * Select the target group: `Ecom-TG`

![Image](https://github.com/user-attachments/assets/c0d645fc-7a7f-4ad7-843e-2f726985df69)
![Image](https://github.com/user-attachments/assets/e5b9447f-ef83-4d8d-baa1-8b3ede378f1d)

6. Configure the group size:

   * **Minimum capacity:** 2 instances
   * **Desired capacity:** 2 instances
   * **Maximum capacity:** 4 instances
7. Configure scaling policy:

   * Choose **Target tracking scaling policy**
   * **Metric type:** Average CPU Utilization
   * **Target value:** 70%
8. Enable CloudWatch group metrics collection.
9. (Optional) Set up notifications:

   * Create an SNS topic (e.g., `Ecom-SNS`)
   * Provide your email to receive alerts
10. Click **Create Auto Scaling group**.

![Image](https://github.com/user-attachments/assets/98667f16-ccf4-4a5f-9aa2-fe35e4428ceb)
---

## 5. 🧪 Validation

* Go to **EC2 > Instances** and verify new instances are created by Auto Scaling.
* Check **Target Group > Targets** for healthy status of the newly added instances.
* Confirm the application is accessible through the **Load Balancer DNS name**.

![Image](https://github.com/user-attachments/assets/88617e24-8bd5-415e-a922-495331316aa9)
![Image](https://github.com/user-attachments/assets/aed7fcca-c3cd-4fbc-80ed-ce8e93f22e1b)
![Image](https://github.com/user-attachments/assets/405e152e-0b38-44ab-96a4-1850185b48bb)
![Image](https://github.com/user-attachments/assets/e8743148-25c2-4af6-9a86-9491e16a80d7)
![Image](https://github.com/user-attachments/assets/e0226844-5ae1-429b-a6ec-0644f204d912)


---

## 5. 🔍 Validate Auto Scaling with Stress Testing

### Step 5.1: Install Stress Tool on One EC2 Instance

SSH into one of the EC2 instances in the Auto Scaling Group and run:

```bash
sudo yum install stress -y
stress --cpu 1 --timeout 10000
```

### Step 5.2: Monitor Scaling Activity

1. Go to **CloudWatch > Alarms** and observe the scaling alarms triggered by high CPU usage.
2. The Auto Scaling group should launch additional EC2 instances based on the policy.
3. Check:

   * **New EC2 instances** in the console
   * **Target group health status** (new instances should show as healthy)
   * **SNS Email Notifications** for scaling activities
![Image](https://github.com/user-attachments/assets/a69a7da1-2807-41f5-96c8-1c1329148b2b)
![Image](https://github.com/user-attachments/assets/a949b88b-a353-40ae-8b42-a01974f1f033)
![Image](https://github.com/user-attachments/assets/de571164-a38c-4e63-b2e9-03e7b2f4d758)
![Image](https://github.com/user-attachments/assets/b81ff6d7-0163-4d54-9d8a-9d8772efc366)
![Image](https://github.com/user-attachments/assets/70d045bd-a184-4cf1-8d80-c5e8848acd34)

---



