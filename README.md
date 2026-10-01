\# AWS Containerized WordPress



A practical cloud and DevOps project demonstrating how to containerize a WordPress application using Docker and prepare it for deployment on AWS.



\## Project Goals



\- Containerize WordPress using Docker

\- Run WordPress and MySQL using Docker Compose

\- Manage configuration using environment variables

\- Keep sensitive credentials out of Git

\- Version control the project using Git and GitHub

\- Deploy the application to AWS

\- Use Amazon ECR for container images

\- Deploy containers using Amazon ECS Fargate

\- Use Amazon RDS MySQL for the database

\- Use Application Load Balancer for traffic distribution

\- Implement IAM, Secrets Manager and CloudWatch

\- Build a CI/CD pipeline



\## Current Architecture



```text

Docker Compose

&#x20;     |

&#x20;     +----------------+

&#x20;     |                |

&#x20;     v                v

WordPress           MySQL

Container           Container

&#x20;     |                |

&#x20;     +------ Network -+

&#x20;            |

&#x20;       Persistent

&#x20;         Volume

