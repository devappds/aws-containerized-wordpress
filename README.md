# AWS Containerized WordPress

A practical AWS Cloud and DevOps project demonstrating how to
containerize a WordPress application with Docker and deploy it on AWS
using Amazon ECS Fargate, Amazon ECR, Application Load Balancer, Amazon
RDS, Amazon VPC, IAM, AWS Secrets Manager, and Amazon CloudWatch.

## Project Overview

This project demonstrates an end-to-end deployment of a containerized
WordPress application on AWS.

The application was first containerized locally using Docker and Docker
Compose. The Docker image was then pushed to Amazon ECR and deployed to
Amazon ECS Fargate behind an internet-facing Application Load Balancer.

The application uses Amazon RDS for MySQL as its database. Database
credentials are stored in AWS Secrets Manager and retrieved by the ECS
task. Networking is isolated with Amazon VPC and Security Groups, while
CloudWatch provides container logs and CPU monitoring.

## Architecture

``` mermaid
flowchart TB
    U[Internet / User] --> ALB[Application Load Balancer<br/>HTTP :80]
    ALB --> TG[Target Group<br/>wordpress-tg]
    TG --> ECS[ECS Fargate<br/>WordPress Container]
    ECS --> RDS[Amazon RDS MySQL<br/>wordpress-db]
    ECS --> SM[AWS Secrets Manager<br/>Database Credentials]
    DEV[Local Docker Project] --> IMG[Docker Image]
    IMG --> ECR[Amazon ECR]
    ECR --> ECS
    ECS --> CW[Amazon CloudWatch<br/>Logs + CPU Alarm]
    ECS --> IAM[IAM Execution Role]
    subgraph VPC[Amazon VPC]
        subgraph PUB[Public Subnets]
            ALB
            ECS
        end
        subgraph PRIV[Private Subnets]
            RDS
        end
    end
    B[AWS Budget<br/>Cost Alerts]
```

## Request Flow

``` text
Internet
   |
   v
Application Load Balancer
   |
   v
Target Group
   |
   v
ECS Fargate
   |
   +----> AWS Secrets Manager
   |
   +----> Amazon RDS MySQL
   |
   +----> CloudWatch Logs
```

## Container Deployment Flow

``` text
Local Docker
     |
     v
Docker Image
     |
     v
Amazon ECR
     |
     v
ECS Fargate
     |
     v
Running WordPress Container
```

## AWS Services Used

| Service                   | Purpose                                                                               |
|---------------------------|---------------------------------------------------------------------------------------|
| Amazon VPC                | Isolated virtual network for the application                                          |
| Subnets                   | Separate public application/load-balancer networking from private database networking |
| Internet Gateway          | Internet connectivity for public resources                                            |
| Route Tables              | Control network routing                                                               |
| Security Groups           | Restrict traffic between ALB, ECS, and RDS                                            |
| Amazon ECR                | Store the Docker container image                                                      |
| Amazon ECS                | Container orchestration                                                               |
| AWS Fargate               | Run containers without managing EC2 servers                                           |
| Application Load Balancer | Receive HTTP traffic and route it to ECS                                              |
| Target Group              | Register ECS task IPs and perform health checks                                       |
| Amazon RDS for MySQL      | Managed WordPress database                                                            |
| AWS Secrets Manager       | Securely store database credentials                                                   |
| IAM                       | Control AWS permissions                                                               |
| Amazon CloudWatch         | Container logs and monitoring                                                         |
| AWS Budgets               | Cost monitoring and budget alerts                                                     |

## Networking

### VPC

A dedicated VPC was created for the application using the `10.0.0.0/16`
CIDR range.

### Public Subnets

Two public subnets were created across two Availability Zones. They are
used for the Application Load Balancer and ECS Fargate task in this
cost-conscious learning setup.

### Private Subnets

Two private subnets were created across two Availability Zones and used
for Amazon RDS. RDS public access is disabled.

### Internet Gateway

An Internet Gateway provides internet connectivity for public subnet
resources.

### Route Tables

The public route table contains:

``` text
0.0.0.0/0 → Internet Gateway
```

The private route table does not contain a default internet route.

## Security Groups

### ALB Security Group

``` text
HTTP :80
Source: 0.0.0.0/0
```

This allows users to reach the internet-facing load balancer.

### ECS Security Group

``` text
HTTP :80
Source: ALB Security Group
```

This limits inbound application traffic to the load balancer.

### RDS Security Group

``` text
MySQL :3306
Source: ECS Security Group
```

This restricts database access to the ECS application layer.

## Docker

The WordPress application was containerized locally using Docker and
Docker Compose.

The repository includes:

``` text
docker-compose.yml
```

Docker provides a consistent application environment and packages the
application into a deployable container image.

## Amazon ECR

The Docker image was pushed to Amazon Elastic Container Registry.

``` text
Docker → Amazon ECR → ECS Fargate
```

ECR acts as the container image registry used by ECS.

## Amazon ECS Fargate

### ECS Cluster

``` text
aws-containerized-wordpress-cluster
```

### ECS Service

``` text
wordpress-service
```

### Task Definition

``` text
aws-containerized-wordpress
```

The task definition defines the container image, CPU, memory, port
mapping, environment variables, secrets, and CloudWatch logging.

Fargate was selected so containers could run without managing EC2
instances.

## Application Load Balancer

An internet-facing Application Load Balancer receives HTTP traffic and
forwards requests to the WordPress ECS task.

``` text
User
  ↓
Application Load Balancer
  ↓
Target Group
  ↓
ECS Fargate Task
```

## Target Group and Health Checks

Target group:

``` text
wordpress-tg
```

Configuration:

``` text
Target type: IP
Protocol: HTTP
Port: 80
```

The health check accepts HTTP status codes:

``` text
200-399
```

This allows successful responses and normal WordPress redirects to be
considered healthy.

## Amazon RDS MySQL

Amazon RDS for MySQL provides the WordPress database.

Configuration:

``` text
Engine: MySQL
Instance class: db.t3.micro
Storage: 20 GiB
Port: 3306
Public access: Disabled
```

The database is placed in private subnets and is not directly exposed to
the public internet.

RDS deletion protection was enabled to reduce the risk of accidental
database deletion.

## AWS Secrets Manager

Database credentials are stored in:

``` text
aws-containerized-wordpress/rds
```

The ECS task uses Secrets Manager values for:

``` text
WORDPRESS_DB_USER
WORDPRESS_DB_PASSWORD
```

The database host, database name, and port are supplied to the WordPress
container as configuration.

This avoids hardcoding the database password in the Docker image or
source code.

> Never commit `.env` files, passwords, access keys, tokens, or other
> credentials to this public repository.

## IAM

The ECS task execution role is used by ECS to access required AWS
resources.

A specific Secrets Manager permission was configured so the ECS task can
retrieve the required database secret.

The project follows the principle of granting only the permissions
required by the workload.

## Amazon CloudWatch

### Container Logs

ECS container logs are sent to:

``` text
/ecs/aws-containerized-wordpress
```

CloudWatch logs were used for deployment troubleshooting and database
connectivity verification.

### CPU Alarm

A CloudWatch alarm was created:

``` text
wordpress-ecs-high-cpu
```

Condition:

``` text
CPU utilization >= 70%
for 5 minutes
```

This provides basic monitoring for unusually high ECS CPU utilization.

## AWS Budget

An AWS Budget was created with a monthly target of:

``` text
$5/month
```

Alerts were configured around:

- 85% actual usage
- 100% actual usage
- Forecasted 100% usage

The budget is used for cost monitoring and does not act as a hard
spending limit.

## Troubleshooting Experience

This project included real AWS deployment troubleshooting.

### ECS / ALB Health Checks

During the initial deployment, ECS tasks failed ELB health checks.

CloudWatch logs and ECS service events were inspected. The target group
health-check matcher was adjusted to:

``` text
200-399
```

After the application and database configuration was corrected, the ECS
service deployed successfully.

### RDS Authentication Error

A later ECS task log showed:

``` text
Access denied for user 'admin'
```

Troubleshooting steps:

1.  Checked ECS task logs in CloudWatch.
2.  Identified the RDS authentication failure.
3.  Updated the RDS master password.
4.  Updated the corresponding Secrets Manager password.
5.  Ran a temporary ECS task to test database access.
6.  Confirmed database creation with `DATABASE_CREATED`.
7.  Updated the WordPress task definition to use the dedicated
    `wordpress` database.
8.  Redeployed the ECS service.
9.  Confirmed the WordPress installation through the Application Load
    Balancer.

This demonstrates practical troubleshooting across ECS, CloudWatch,
Secrets Manager, Security Groups, and RDS.

## Project Structure

``` text
aws-containerized-wordpress/
│
├── .gitignore
├── README.md
└── docker-compose.yml
```

## Local Development

Clone the repository:

``` bash
git clone https://github.com/devappds/aws-containerized-wordpress.git
```

Move into the project:

``` bash
cd aws-containerized-wordpress
```

Start the local environment:

``` bash
docker compose up -d
```

Stop the environment:

``` bash
docker compose down
```

For local database credentials, use an environment file excluded from
Git with `.gitignore`.

## Deployment Workflow

``` text
1. Develop and test WordPress locally
              |
              v
2. Build the Docker image
              |
              v
3. Push image to Amazon ECR
              |
              v
4. Create VPC, subnets, route tables and security groups
              |
              v
5. Create Amazon RDS MySQL
              |
              v
6. Store database credentials in Secrets Manager
              |
              v
7. Create ECS task definition
              |
              v
8. Deploy WordPress on ECS Fargate
              |
              v
9. Configure Application Load Balancer
              |
              v
10. Configure target-group health checks
              |
              v
11. Configure CloudWatch logs and CPU monitoring
              |
              v
12. Validate WordPress through the ALB
```

## Security Practices

- RDS public access disabled
- RDS deployed in private subnets
- RDS access restricted to the ECS security group
- ECS inbound traffic restricted to the ALB security group
- Database password stored in Secrets Manager
- IAM permissions used for AWS resource access
- RDS deletion protection enabled
- CloudWatch logging enabled
- `.env` excluded from Git

## What I Learned

This project provided hands-on experience with:

- Docker and containerization
- Git and GitHub
- Amazon ECR
- Amazon ECS
- AWS Fargate
- Application Load Balancer
- Target groups and health checks
- Amazon VPC
- Public and private subnets
- Route tables and Internet Gateway
- Security Groups
- Amazon RDS MySQL
- AWS Secrets Manager
- IAM permissions
- CloudWatch Logs
- CloudWatch alarms
- AWS Budgets
- AWS troubleshooting and deployment debugging

## Interview Summary

> I built and deployed a containerized WordPress application on AWS. I
> created a custom VPC with public and private subnets, deployed the
> Docker image to Amazon ECR, ran WordPress on ECS Fargate, placed an
> Application Load Balancer in front of the application, connected
> WordPress to a private Amazon RDS MySQL database, secured database
> credentials using AWS Secrets Manager and IAM, and configured
> CloudWatch logs and CPU monitoring. I also troubleshot real ECS/ALB
> health-check and RDS authentication issues during deployment.

## Future Improvements

Potential production-oriented improvements include:

- HTTPS using AWS Certificate Manager
- Custom domain using Amazon Route 53
- CI/CD using GitHub Actions
- Automated Docker image builds
- ECS service auto scaling
- CloudWatch dashboards
- Infrastructure as Code using Terraform or AWS CloudFormation
- Blue/green deployments
- Multi-AZ RDS configuration
- Persistent WordPress media storage using Amazon S3/EFS
- More comprehensive application monitoring

## Repository

GitHub: https://github.com/devappds/aws-containerized-wordpress
