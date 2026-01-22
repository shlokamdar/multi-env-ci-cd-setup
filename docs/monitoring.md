# Monitoring Strategy

## Objective

To monitor Production infrastructure health while avoiding unnecessary overhead on UAT systems.

---

## Monitoring Stack

- Amazon CloudWatch Agent
- CPU, memory, and disk metrics
- Nginx service monitoring

---

## Deployment Strategy

- CloudWatch Agent installed only on Production node
- Automated via Ansible


