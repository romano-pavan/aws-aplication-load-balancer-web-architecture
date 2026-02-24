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

sudo su
dnf update -y
dnf install httpd -y
systemctl enable --now httpd
echo "<h1>Hello! I am super safe server from Private subnet</h1>" > /var/www/html/index.html

5.Set up the ALB, attached the target group, and routed the DNS name to my server

![ALB](images/load-balancer.jpg)

## Proof of Concept

Web server successfully responding through the Load Balancer's public DNS, proving the private-to-public routing works:




