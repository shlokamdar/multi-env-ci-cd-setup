# Architecture Overview

## High-Level Architecture

This project follows a **master–agent Jenkins architecture** deployed within a secure AWS VPC.  
The Jenkins master orchestrates builds while deployments are executed on isolated environment-specific agents.

---

## Architecture Diagram

![Architecture Diagram](../diagrams/architecture.png)


---

## Architecture Explanation

- Jenkins Master:
  - Handles pipeline orchestration
  - Manages credentials and multibranch indexing
- Jenkins Agents:
  - UAT agent handles `develop` branch deployments
  - Production agent handles `main` branch deployments
- Application Load Balancer:
  - Acts as the single public entry point
  - Routes traffic to healthy instances
- CloudWatch:
  - Enabled only on Production for monitoring system metrics

---

## Benefits of This Architecture

- Environment isolation
- Scalable CI/CD execution
- Secure and controlled access
- Production-grade observability
