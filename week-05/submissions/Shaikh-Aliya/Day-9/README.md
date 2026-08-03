# Day 9 - ALB Backed Auto Scaling Group

## Objective

The objective of this lab was to build a highly available and scalable web application architecture using an Application Load Balancer (ALB) and Auto Scaling Group (ASG).

The infrastructure automatically distributes incoming traffic across EC2 instances and launches or terminates instances based on CPU utilization.

---

## AWS Services Used

- Amazon EC2
- Launch Template
- Security Groups
- Target Group
- Application Load Balancer (ALB)
- Auto Scaling Group (ASG)
- CloudWatch
- IAM
- Amazon Linux 2023
- Nginx

---

## Resources Created

| Resource | Name |
|----------|------|
| Region | ap-south-1 |
| ALB Security Group | cloudadhar-day9-alb-sg |
| Web Security Group | cloudadhar-day9-web-sg |
| Launch Template | cloudadhar-day9-lt |
| Target Group | cloudadhar-day9-tg |
| Application Load Balancer | cloudadhar-day9-alb |
| Auto Scaling Group | cloudadhar-day9-asg |
| Scaling Policy | cloudadhar-day9-cpu50-policy |
| EC2 Instance | cloudadhar-day9-web-instance |

---

## Tags Used

The following tags were applied to all supported resources.

| Key | Value |
|------|-------|
| Project | AWS-Zero-To-Hero |
| Day | 09 |
| Environment | Training |
| Owner | CloudAdhar |
| DataClassification | Training-Only |

The Auto Scaling Group was configured to propagate these tags automatically to every new EC2 instance.

---

# Step 1 - Create Security Groups

Two security groups were created to separate internet traffic from application traffic.

### ALB Security Group

Configured to allow:

- HTTP (Port 80) from anywhere (`0.0.0.0/0`)

Purpose:

- Accept incoming requests from users on the internet.

### Web Server Security Group

Configured to allow:

- HTTP (Port 80) **only** from the ALB Security Group.
- SSH (Port 22) temporarily from my public IP for administration.

Purpose:

- Ensure EC2 instances accept web traffic only from the load balancer.

### Screenshots

![ALB Security Group](Screenshots/sg-alb.png)

![Web Security Group](Screenshots/sg-web.png)

---

# Step 2 - Create Launch Template

A Launch Template was created to define the configuration used whenever the Auto Scaling Group launches a new EC2 instance.

Configuration included:

- Amazon Linux 2023
- t3.micro
- 8 GiB gp3 EBS volume
- IMDSv2 enabled
- Detailed Monitoring enabled
- Web Security Group attached
- IAM Role attached

User Data was added to automatically:

- Install Nginx
- Enable and start the service
- Retrieve EC2 metadata using IMDSv2
- Create a dynamic web page displaying:
  - Instance ID
  - Instance Type
  - Availability Zone
  - Private IP
- Create `/health.html` for ALB health checks

This allows every new EC2 instance launched by the Auto Scaling Group to configure itself automatically without manual intervention.

### Screenshot

![Launch Template](Screenshots/instance1-launch-template.png)

---

# Step 3 - Create Target Group

An Instance Target Group was created.

Configuration:

- Target Type: Instance
- Protocol: HTTP
- Port: 80
- Health Check Path: `/health.html`
- Success Code: 200

Purpose:

The Target Group continuously checks whether each EC2 instance is healthy before sending user traffic.

### Screenshot

![Target Group](Screenshots/tg.png)

---

# Step 4 - Create Application Load Balancer

An internet-facing Application Load Balancer was created.

Configuration:

- Listener: HTTP (80)
- Forward requests to the Target Group
- Two Availability Zones selected
- ALB Security Group attached

Purpose:

The ALB distributes incoming requests across healthy EC2 instances and improves application availability.

### Screenshot

![Application Load Balancer](Screenshots/lb.png)

---

# Step 5 - Create Auto Scaling Group

The Auto Scaling Group was created using the Launch Template.

Configuration:

- Launch Template attached
- Existing Target Group attached
- Minimum Capacity: 1
- Desired Capacity: 1
- Maximum Capacity: 2
- Health Check Type:
  - EC2
  - ELB
- Grace Period: 300 seconds
- Instance Warmup: 120 seconds

Purpose:

The Auto Scaling Group maintains the desired number of EC2 instances and automatically replaces unhealthy instances.

### Screenshot

![Auto Scaling Group](Screenshots/ASG.png)

---

# Step 6 - Verify Initial Deployment

After the Auto Scaling Group launched the first EC2 instance:

- Instance became healthy.
- Target Group reported the target as Healthy.
- Application became accessible through the ALB DNS.

The following command was used to verify the ALB response:

```bash
curl http://<ALB-DNS>
```

### Screenshot

![Curl Verification](Screenshots/curl-command.png)

---

# Step 7 - Configure Dynamic Scaling Policy

A Target Tracking Scaling Policy was created.

Configuration:

- Metric: Average CPU Utilization
- Target Value: 50%
- Scale-In Enabled

Purpose:

Whenever average CPU utilization exceeds 50%, the Auto Scaling Group launches another EC2 instance.

When CPU utilization falls below the target value, the extra instance is automatically terminated.

### Screenshot

![Scaling Policy](Screenshots/ASG-desiredstatecomplete.png)

---

# Step 8 - Generate CPU Load

To test automatic scaling, CPU stress was generated using `stress-ng`.

Commands:

```bash
sudo dnf install -y stress-ng

nohup stress-ng --cpu 2 --cpu-load 95 --timeout 10m \
> /tmp/stress-ng.log 2>&1 &

pgrep -af stress-ng
```

Purpose:

The workload increased CPU utilization above the configured threshold, triggering the Auto Scaling policy.

### Screenshot

![CPU Load](Screenshots/load-before-&-after-terminal.png)

---

# Step 9 - Verify Scale Out

After CPU utilization remained above the configured threshold:

- CloudWatch Alarm entered the ALARM state.
- Desired Capacity changed from 1 to 2.
- Auto Scaling Group launched a new EC2 instance.
- The new instance automatically joined the Target Group.
- Health checks passed successfully.

### Screenshot

![Desired Capacity Increased](Screenshots/desiredstatewaiting-ASG.png)

### Screenshot

![Second Instance Created](Screenshots/2nd-instance-launched.png)

---

# Step 10 - Verify Load Balancing

After the second instance became healthy, repeated requests to the ALB returned responses from different EC2 instances.

The displayed Instance ID changed between requests, confirming that traffic was distributed across both instances.

### Screenshot

![Instance 1 Response](Screenshots/servingroleto-instanceid1.png)

### Screenshot

![Instance 2 Response](Screenshots/serving-role-instanceid2.png)

---

# Step 11 - Verify Scale In

After stopping the CPU stress process:

```bash
sudo pkill stress-ng

pgrep -af stress-ng
```

CPU utilization gradually decreased.

Once CloudWatch detected sustained low CPU usage:

- Desired Capacity changed from 2 to 1.
- One EC2 instance entered the Draining state.
- Auto Scaling Group terminated the extra instance.
- The remaining instance continued serving requests.

This confirmed that automatic scale-in was functioning correctly.

### Screenshot

![Scale In Completed](Screenshots/ASG-desiredstatecomplete.png)

---

# Troubleshooting

### Unable to access the application using EC2 Public IP

Issue:

The application was reachable through the ALB DNS but not directly through the EC2 Public IP.

Root Cause:

The Web Security Group allowed HTTP traffic only from the ALB Security Group, which intentionally blocks direct internet access to EC2.

Resolution:

Verified application access using the ALB DNS, which is the intended production architecture.

---

### SSH Connection Failed

Issue:

Unable to connect to the EC2 instance.

Root Cause:

SSH rule had been removed from the Web Security Group.

Resolution:

Temporarily added an SSH rule for my public IP, connected successfully, completed verification, and removed the rule afterward.

---

### Auto Scaling Did Not Launch a New Instance Immediately

Issue:

The second EC2 instance was not created immediately after starting CPU stress.

Root Cause:

The Target Tracking policy waits for CloudWatch metrics and alarm evaluation before scaling.

Resolution:

Continued generating CPU load until the CloudWatch Alarm entered the ALARM state, after which the Auto Scaling Group launched a second instance.

---

## Learning Outcomes

Through this lab I learned how to:

- Configure secure communication between ALB and EC2 using Security Groups.
- Create reusable Launch Templates.
- Bootstrap EC2 instances using User Data.
- Configure ALB health checks.
- Build an Auto Scaling Group using a Launch Template.
- Configure Target Tracking Scaling Policies.
- Trigger automatic scale-out based on CPU utilization.
- Verify ALB traffic distribution across multiple instances.
- Observe automatic scale-in after CPU utilization decreased.
- Troubleshoot common issues related to Security Groups, ALB health checks, and Auto Scaling behavior.
