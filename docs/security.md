# Security Design

## Security Principles Applied

- Least privilege access
- Environment isolation
- No direct public EC2 access
- Centralized entry via ALB

---

## Security Groups

- Jenkins UI restricted to trusted IP
- SSH allowed only from Jenkins master
- Application traffic allowed only via ALB

📸 **Placeholder: Security Group Rules**


---

## Bastion Strategy

The Jenkins master acts as a bastion host to control access to internal EC2 instances securely.
