
# Project Overview

## Objective

To design and implement a **branch-based CI/CD pipeline** using Jenkins that deploys a single GitHub repository to **separate UAT and Production environments** on AWS, using **dedicated Jenkins agents**, with **monitoring enabled on the production environment**.

---

## Components Used
![Architecture Diagram](diagrams/architecture.png)
- AWS VPC
- Jenkins Master (EC2)
- Jenkins Agent – UAT (EC2)
- Jenkins Agent – Production (EC2)
- GitHub Repository (single repo, multiple branches)
- Nginx Web Server
- AWS Application Load Balancer (ALB)
- Amazon CloudWatch (Production only)
- Ansible (Configuration Management)

---

## Environment Strategy

| Environment | Git Branch | Jenkins Agent | URL |
|------------|-----------|---------------|-----|
| UAT | `develop` | `uat-slave` | `uat.domain.in` |
| Production | `main` | `prod-slave` | `domain.in` |

---

## Key Design Decisions

- Separate EC2 instances per environment to ensure **isolation and stability**
- Jenkins master used only for orchestration
- Environment-specific Jenkins agents for deployments
- Monitoring enabled only on Production to reduce overhead
