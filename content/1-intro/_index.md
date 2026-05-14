---
title : "Introduction"
date: 2024-01-01
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Introduce

#### The script
We will proceed to integrate the **Secrets Manager** service with **Amazon RDS** and **AWS Fargate**.
1. The first part of the exercise aims to access **RDS Database** which will be managed through **Secrets Manager**.
2. In the second part, we go through the **Secret Rotation** process to change the password periodically and do some tests with Shell Scripts.
3. Last part, we will try accessing **RDS Database** from the application layer with **Fargate Container**.

This exercise will require some knowledge of **Docker** as well as **Linux**.
- Level: 300
- Duration: 1-2 hours

#### Prerequisites:
  - AWS account
  - IAM User (Admin)
  - Knowledge of Linux
  - Knowledge of Docker and AWS Fargate

#### Functions of **CSF** (Cybersecurity Framework):
  - Prevention (Prevent)
#### Security Perspectives of **CAF** (Cloud Adoption Framework):
  - Ability to prevent (Preventative)

#### AWS Services Used:
  - Amazon EC2
  - Amazon RDS
  - Amazon ECS
  - Amazon ECR
  - AWS Secrets Manager
  - Amazon VPC

#### Architectural models
The network architecture model will be a VPC with 2 Subnets. In it, a **Bastion Host** based on *Amazon Linux 2* operating system, will be used to run shell scripts and AWS CLI. Also, a **Private RDS Instance** is the main goal of the exercise, because we will be using **RDS credential** storage with the **Secrets Manager** service. In addition, we will deploy **AWS Fargate** to be able to test the connection to **RDS database** and from **EC2 instance** and **Fargate Container**.

![workshop-architecture](/images/1-workshop-architecture.png?featherlight=false&width=60p)