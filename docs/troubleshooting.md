# Troubleshooting & Lessons Learned

## Jenkins Agent Failed to Connect

**Issue:** Jenkins agent failed during launch  
**Cause:** Java not installed on agent  
**Resolution:** Installed Java using Ansible

---

## Pipeline Failed During Build

**Issue:** `npm: command not found`  
**Cause:** Node.js not installed on agent  
**Resolution:** Installed Node.js via Ansible

---

## UAT Stages Skipped

**Reason:** Expected behavior when pipeline runs on `main` branch

---

## Key Takeaways

- Jenkins agents must have all runtime dependencies
- Automation prevents repeated configuration errors
- Observability should focus on production workloads
