# CI/CD Pipeline Flow

## Overview

This pipeline uses **Jenkins Multibranch Pipelines** to automate deployments based on Git branch strategy.

---

## Pipeline Flow

1. Code pushed to GitHub
2. Jenkins indexes branches
3. Jenkinsfile detected
4. Agent selected based on branch
5. Build and deployment executed
6. Application served via Nginx
7. Production monitored via CloudWatch

---

## Branch-to-Environment Mapping

| Branch | Environment | Agent |
|------|-------------|-------|
| develop | UAT | uat-slave |
| main | Production | prod-slave |

---
![Multibranch Pipeline](diagrams/success-mbp.png)


