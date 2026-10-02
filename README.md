# AWS Containerized WordPress

A practical AWS Cloud and DevOps project demonstrating how to containerize a WordPress application with Docker and deploy it on AWS using Amazon ECS Fargate, Amazon ECR, Application Load Balancer, Amazon RDS, VPC, IAM, Secrets Manager, and CloudWatch.

---

## 🚀 Project Overview

This project demonstrates the deployment of a containerized WordPress application on AWS.

The application was first containerized locally using Docker and then deployed to AWS ECS Fargate.

The architecture separates the application, database, networking, security, secrets, and monitoring layers.

---

## 🏗️ Architecture

```text
                         Internet
                            |
                            v
                +------------------------+
                | Application Load       |
                | Balancer (ALB)         |
                | HTTP : 80               |
                +-----------+------------+
                            |
                            v
                +------------------------+
                | Amazon ECS Fargate     |
                | WordPress Container    |
                | Port 80                |
                +-----------+------------+
                            |
                            |
                  +---------+---------+
                  |                   |
                  v                   v
        +----------------+   +----------------------+
        | AWS Secrets    |   | Amazon RDS MySQL     |
        | Manager        |   | wordpress-db         |
        | DB Credentials |   | Port 3306            |
        +----------------+   +----------------------+

        Docker
           |
           v
    Amazon ECR
           |
           v
    ECS Fargate


        VPC
        |
        +-- Public Subnets
        |      +-- ALB
        |      +-- ECS
        |
        +-- Private Subnets
               +-- RDS

        CloudWatch
        +-- ECS Logs
        +-- CPU Alarm

        IAM
        +-- ECS Task Permissions

        AWS Budget
        +-- Cost Monitoring
