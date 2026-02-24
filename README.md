# aws-aplication-load-balancer-web-architecture
building a highly available and secure web architecture in AWS using ALB and Private Subnets

About project

Get experience with AWS networking and security. My main goal was to answer a common infrastructure question: "How do I serve a website to the public internet without actually exposing my web servers to hackers?"
To solve this, I designed a multi-tier VPC architecture where the Application Load Balancer (ALB) sits in the public subnet to handle incoming traffic, while the actual Apache web server is safely isolated in a private subnet with no public IP address.

## Architecture Design
![AWS Architecture Diagram](images/architecture-diagram.png)

## Tech Stack
**Networking:** Amazon VPC, Internet Gateway, NAT Gateway, Route Tables.

**Compute & Routing:** Amazon EC2 (Amazon Linux 2023), Application Load Balancer (ALB), Target Groups.

**Security:** AWS IAM (Roles for SSM), strictly chained Security Groups.

**Management:** AWS Systems Manager (SSM) Session Manager (Used for secure, keyless terminal access).

## Setup Process

1. Created a custom VPC (10.0.0.0/16) with public and private subnets across two different Availability Zones to ensure high availability for the Load Balancer.
   
2. Configured the IGW for public subnets and a NAT Gateway so my private server could securely download the Apache packages.


![VPC](images/my-safe-vpc.jpg)

3. Configured the EC2 instance's security group to only accept HTTP traffic if it physically comes from the ALB's security group. Direct access from the outside is completely blocked.

![SG](images/inbound-security-group.jpg)

4.Deployed the EC2 instance, connected via SSM (no SSH keys needed!), installed Apache, and created a simple HTML page.

```sudo su```        
```dnf update -y```             
```dnf install httpd -y```
```systemctl enable --now httpd```
```echo "Hello! I am super safe server from Private subnet"> /var/www/html/index.html ```


5.Set up the ALB, attached the target group, and routed the DNS name to my server

![ALB](images/load-balancer.jpg)

## Proof of Concept

Web server successfully responding through the Load Balancer's public DNS, proving the private-to-public routing works:

![website](images/accesed-website.jpg)


## Troubleshooting & Lessons Learned

At one point, I deleted my NAT Gateway to save costs. When I recreated it later, my EC2 instance completely dropped offline in the AWS SSM console. It kept throwing a `RequestError: send request failed` error.

I realized that recreating the NAT Gateway generated a new ID. My Private Route Table was still pointing to the old, deleted NAT ID (creating a blackhole route). Once I updated the route table with the new NAT Gateway ID, the connection was still hanging. A quick reboot of the EC2 instance forced the SSM service to restart, allowing it to finally regain internet access and reconnect to the AWS API.







