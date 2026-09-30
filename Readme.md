# Secure Private AWS Infrastructure with Bastion & IAM

In this Project, we will Building a Secure Private Infrastructure Deployment with Bastion Access and IAM Role-Based Architecture.

## 1. Project Overview

This project demonstrates how to deploy a secure AWS infrastructure where the application server runs inside a private subnet and cannot be accessed directly from the internet.

A Bastion Host is used as the controlled SSH entry point to access the private EC2 instance.

The private EC2 instance uses an IAM Role to access Amazon S3 without storing AWS access keys on the server.


## Main Objectives
* Deploy application servers in private subnets.
* Prevent direct public access to the private EC2 instance.
* Use a Bastion Host for SSH access.
* Restrict SSH access using Security Groups.
* Use IAM Roles instead of hardcoded AWS credentials.
* Verify S3 access from the private EC2 instance.

## 2. Architecture
__Architecture Diagram__ 

![src1](./img/ChatGPT%20Image%20Sep%2030,%202026,%2006_07_05%20PM.png)

__Architecture Explanation__

The project uses a custom AWS VPC `(10.0.0.0/16)` with two public and two private subnets.
The public subnets contain the Bastion Host and NAT Gateway, while the application server is placed in a private subnet.
The public subnets use an Internet Gateway for internet connectivity, while the private subnets use the NAT Gateway for required outbound access.
The private EC2 instance has no public IP, which prevents direct access from the internet.

The Bastion Host acts as the secure entry point for SSH access to the private EC2 instance.
Security Groups allow SSH access to the Bastion only from the user's IP and allow the private EC2 to accept SSH only from the Bastion.
The private EC2 runs Nginx and uses the `EC2-S3-ReadOnly-Role` IAM role for limited S3 access without storing AWS access keys.

This design provides network isolation, controlled SSH access, and role-based AWS access.

## 3. Requirements
* Amazon VPC
* Amazon EC2
* Amazon S3
* AWS IAM
* Internet Gateway
* NAT Gateway
* Security Groups
* Route Tables
* Nginx/Appache webserver 

## 4. Steps
### 1. Create VPC
* VPC CIDR: `10.0.0.0/16`
### 2. Create Subnets
* Public Subnet 1: `10.0.3.0/24`
* Public Subnet 2: `10.0.6.0/24`
* Private Subnet 1: `10.0.9.0/24`
* Private Subnet 2: `10.0.12.0/24`
### 3. Create Internet Gateway (IGW)
* Create IGW.

* Attach it to the VPC.

* Used by public subnets for internet connectivity.
### 4. Create NAT Gateway
* Create NAT Gateway in a public subnet.

* Used by private subnets for outbound internet access.
### 5. Create Route Tables
* Public Route Table: `0.0.0.0/0` → IGW
* Private Route Table: `0.0.0.0/0` → NAT Gateway
* Associate the appropriate subnets.

## 2. Bastion Server Creation
### 1. Create Bastion Security Group
* Allow SSH `(22)` only from your public IP.
* Do not allow SSH from `0.0.0.0/0`

![src2](./img/Screenshot%202026-09-30%20110326.png)
### 2. Launch Bastion EC2
* Launch EC2 in a Public Subnet.
* Enable public IP.
* Select the required key pair.

![src4](./img/Screenshot%202026-09-30%20104340.png)
### 3. Configure SSH Access
* Use key-based authentication.
* Connect from the local machine to the Bastion.

## 3. Private Server Creation
### 1. Create Private Server Security Group
* Allow SSH `(22)` only from the __Bastion Security Group__.
* Do not allow direct SSH from the internet.

![src3](./img/Screenshot%202026-09-30%20103732.png)
### 2. Launch Private EC2
* Launch EC2 in a Private Subnet.
* Disable public IP.
* Select the required key pair.
* Select the Security group that allow bastion server only

![src5](./img/Screenshot%202026-09-30%20104539.png)
### 3. Connect Through Bastion
```
Local Machine
      ↓ SSH
Bastion Server
      ↓ SSH
Private Server
```

![sec6](./img/Screenshot%202026-09-30%20140829.png)
### 4. Install Nginx

* Install and start Nginx on the private server.

* Sample page:

__`This is From Private Server`__

## 4. IAM Role Configuration
### 1. Create IAM Role

* Create:

`EC2-S3-ReadOnly-Role`

### 2. Attach S3 Read Permission
* Attach limited S3 read-only permission to the role.

![src8](./img/Screenshot%202026-09-30%20105155.png)
### 3. Attach Role to Private EC2
* Attach EC2-S3-ReadOnly-Role to the private EC2 instance.
![src9](./img/Screenshot%202026-09-30%20105303.png)
### 4. Verify S3 Access

* From the private server:

      `aws s3 ls`


 ![src7](./img/Screenshot%202026-09-30%20143119.png)

* Verify that S3 access works without storing AWS access keys on the EC2 instance.

## 5. Security Design Decisions
* Private EC2:- The application server is placed in a private subnet with no public IP.
* Bastion Host:- SSH access to the private server is allowed only through the Bastion Host.
* Restricted SSH:- Bastion SSH is allowed only from the user's IP address.
* Security Groups:- Separate Security Groups are used to control access between the Bastion and private server.
* Private Routing:- Private subnets do not have a direct route to the Internet Gateway.
* NAT Gateway:- Used to provide required outbound internet access from the private subnet.
* IAM Role:- The private EC2 uses EC2-S3-ReadOnly-Role instead of storing AWS access keys.
* Least Privilege:- The IAM role provides only the required S3 read permissions.

## 6. Lessons Learned

Through this project, I learned:

1. How to create and configure a custom AWS VPC, public and private subnets creation.
2. How Internet Gateway and NAT Gateway are used in a VPC.
3. How route tables control network traffic.
4. How to use a Bastion Host to access private EC2 instances.
5. How Security Groups can restrict SSH access.
6. How to attach an IAM Role to an EC2 instance.
7. How to access Amazon S3 without storing AWS access keys on the server.

## 7. Project Summary

This project focuses on designing a secure AWS infrastructure for a fintech application by removing direct internet exposure of application servers. A custom VPC is created with public and private subnets, along with an Internet Gateway, NAT Gateway, and appropriate routing.

A Bastion Host is deployed in the public subnet to provide controlled SSH access to the private application server. The private EC2 instance has no public IP and runs the application using Nginx. Security Groups are configured to restrict SSH access and maintain network isolation.

An IAM Role with limited S3 read permissions is attached to the private EC2 instance, allowing it to access S3 without storing AWS access keys. The project validates that the private server cannot be accessed directly, SSH access works through the Bastion Host, and S3 access works through the IAM role.