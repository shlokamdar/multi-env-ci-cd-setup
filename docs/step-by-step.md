This section documents the end-to-end infrastructure and CI/CD setup process, starting from network creation to Jenkins-based deployments across UAT and Production environments.

The goal is to demonstrate how each AWS and DevOps component is provisioned, configured, and connected, ensuring environment isolation, deployment safety, and operational clarity.

# 1. Setting up a VPC

**Purpose**

To create a secure and isolated AWS network that hosts all CI/CD and application components, enabling controlled access and scalability.

**Actions Performed**

- Created a custom VPC
- Configured public subnets for:
    - Bastion/Master node, Jenkins Agents, Load Balancer
- Configured route tables and internet gateway
- Applied security groups for SSH, HTTP, and Jenkins access

![VPC architecture diagram](../diagrams/vpc-resource-map.png)

# 2. Setting Up Compute Components (EC2 Instances)

**Purpose**

To provision dedicated compute resources for Jenkins orchestration, environment-specific deployments, and monitoring.

**Instances Launched**

- Jenkins Master / Bastion Host
- Jenkins Agent – UAT
- Jenkins Agent – Production

![EC2 instances list](../diagrams/ec2-isntances.png)

# 3. Bastion/Jenkins Master node

**Role**

This node acts as:

- **Jenkins Master** for CI/CD orchestration
- **Ansible Control Node** for configuration management and monitoring setup

### **Jenkins Installation**

```bash
# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install dependencies
sudo dnf upgrade -y
sudo dnf install java-17-amazon-corretto -y
sudo dnf install jenkins -y

# Start Jenkins
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Verify it on `<public-ip>:8080`

![Jenkins UI](../diagrams/jenkins-ui.png)

### **Ansible Installation & Configuration**

**Purpose —** To centrally manage Jenkins agents and install required services (Nginx, Git, monitoring tools)

```bash
#!/bin/bash
set -e

# Create ansible user
useradd ansible || true
mkdir -p /home/ansible/.ssh

# Set permissions
chown -R ansible:ansible /home/ansible
chmod 700 /home/ansible/.ssh

# Enable passwordless sudo
echo "ansible ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible
chmod 440 /etc/sudoers.d/ansible

# Enable SSH password authentication
sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Disable root login
sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

systemctl restart sshd

# Install Ansible
sudo dnf install ansible -y
```

- Generated SSH keys on master
- Copied keys to agent nodes
- Enabled passwordless SSH access

```bash
ssh-keygen
ssh-copy-id ansible@<agent-ip>
# Manually ssh into slaves to set ansible password but that can be added in the setup script as well
```

### Ansible playbook to install prerequisites

- Nginx - webservers
- Git
- Java - Jenkins dependency

```yaml
- name: Install and configure Nginx, Git, and Java on web servers
  hosts: webservers
  become: yes

  tasks:
    - name: Install Java (required for Jenkins agent)
      yum:
        name: java-17-amazon-corretto
        state: present

    - name: Install nginx
      yum:
        name: nginx
        state: present

    - name: Install git
      yum:
        name: git
        state: present

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

![Ansible Playbook Execution Output](../diagrams/ansible-playbook-success.png)

# 4. Jenkins Agent – Production Environment

**Role**

- Handles deployments triggered from the `main` branch
- Hosts production-grade Nginx application
- Connected to Application Load Balancer
- Integrated with monitoring stack

# 5. Jenkins Agent – UAT Environment

**Role**

- Handles deployments triggered from the `develop` branch
- Used for testing and validation before production
- Isolated from production infrastructure

# 6. Security Groups Configured

**Objective -**

To enforce **least-privilege network access** by explicitly controlling inbound traffic between the Jenkins master, agents, load balancer, and public users—while preventing unnecessary exposure of internal components.

### **Jenkins Master/ Bastion Security Group**

**Purpose -** Allows administrative access and CI/CD orchestration while restricting Jenkins UI access to trusted sources.

| Protocol  | Port | Source |
| --- | --- | --- |
| SSH | 22 | My IP / Trusted IP |
| Jenkins UI | 8080 | My IP |
| HTTP (optional) | 80 | ALB Security Group |

### Jenkins Agents - UAT Security Group & Production Security Group

Allows Jenkins master to trigger builds and deploy artifacts while exposing the application only via the load balancer.

| Rule Name | Port | Source |
| --- | --- | --- |
| SSH | 22 | Jenkins Master SG |
| HTTP | 80 | ALB Security Group |
| HTTPS  | 443  | ALB Security Group |

### Application Load Balancer Security Group

Allows public HTTP access while acting as the **only public-facing entry point** to the application.

| Rule Name | Port | Source |
| --- | --- | --- |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

# 7. Application Load Balancer Setup

**Objective**

To distribute incoming HTTP traffic across environment-specific EC2 instances while enabling high availability, fault tolerance, and clean separation between UAT and Production environments.

### Target Group Configuration

Target groups define **where the ALB forwards traffic** and allow environment-level separation.

**Target Groups Created:**

| Target Group | Environment | Protocol | Port | Health Check |
| --- | --- | --- | --- | --- |
| `tg-uat` | UAT | HTTP | 80 | `/` |
| `tg-prod` | Production | HTTP | 80 | `/` |
- Targets registered: Respective EC2 instance (Jenkins agents)
- Health checks ensure traffic is sent only to healthy nodes

### Application Load Balancer (ALB)

Created two ALBs - one for production and one for UAT with same configuration just different TG

**Configuration**

- Type: Application Load Balancer
- Scheme: Internet-facing
- Listener: HTTP (Port 80)
- Security Group: ALB Security Group
- Subnets: Public Subnets

**Listener Rules**

| Listener Port | Action | Target Group |
| --- | --- | --- |
| 80 | Forward | UAT or Production TG |
| 443  | Forward | UAT or Production TG |

# Jenkins Setup & GitHub Integration for CI/CD

**Objective -**

To configure Jenkins as a **central CI/CD orchestrator**, integrate it with GitHub as the source control system, register environment-specific build agents, and validate end-to-end pipeline execution using a **multi-branch pipeline strategy**.

**Accessing Jenkins -**

Jenkins was exposed on the Jenkins Master EC2 instance using its public IP.

Jenkins URL —

```
http://<JENKINS_PUBLIC_IP>:8080
```

**Initial Admin Password Retrieval**

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

This password was used to:

- Unlock Jenkins
- Install recommended plugins
- Create the initial admin user

![Jenkins UI](../diagrams/jenkins-ui.png)

### Jenkins–GitHub Integration

**Objective**

To allow Jenkins to automatically pull source code from a **single GitHub repository** containing multiple branches for different environments.

**Configuration Steps**

Manage Jenkins → Credentials → Add Credentials

- Generated a **GitHub Personal Access Token (PAT)**
- Added GitHub credentials in Jenkins:
    - Type: Secret Text
    - Scope: Global
- Configured GitHub repository in Jenkins

![Configuring GitHub](../diagrams/configuring-github.png)

### Jenkins Agent (Node) Configuration

**Objective**

To offload builds and deployments to **dedicated environment-specific agents**, ensuring isolation between UAT and Production.

**Nodes configured**

| Node Name | Environment | Label |
| --- | --- | --- |
| `slave-uat` | UAT | `uat-slave` |
| `slave-prod` | Production | `prod-slave` |

![List of Nodes](../diagrams/list-of-nodes.png)

**Steps taken:**

Manage Jenkins → Nodes→ Add Node

