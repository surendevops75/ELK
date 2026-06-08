# ELK Stack Installation and Configuration Guide

This guide demonstrates how to install and configure a complete ELK Stack environment for centralized logging.

The setup includes:

- Elasticsearch
- Kibana
- Logstash
- Filebeat
- Nginx Reverse Proxy
- Kubernetes (EKS) Log Collection

This architecture enables centralized log management, monitoring, troubleshooting, and observability across servers and Kubernetes workloads.

---

# ELK Architecture Overview

```text
Applications
     │
     ▼
Filebeat
     │
     ▼
Logstash
     │
     ▼
Elasticsearch
     │
     ▼
Kibana
```

For Kubernetes:

```text
Pods
  │
  ▼
Filebeat DaemonSet
  │
  ▼
Logstash
  │
  ▼
Elasticsearch
  │
  ▼
Kibana
```

---

# Prerequisites

Before installing ELK:

- Amazon Linux / RHEL / Rocky Linux
- Minimum 60 GB disk space
- Internet access
- Root or sudo privileges

---

# Disk Expansion

Elasticsearch requires significant storage.

Expand the root disk partition:

```bash
sudo growpart /dev/nvme0n1 4
```

Extend logical volumes:

```bash
sudo lvextend -L +20G /dev/RootVG/rootVol
sudo lvextend -L +20G /dev/RootVG/varVol
```

Resize filesystems:

```bash
sudo xfs_growfs /
```

```bash
sudo xfs_growfs /var
```

Verify:

```bash
df -h
```

Recommended:

```text
Root Volume  : 30 GB+
Var Volume   : 30 GB+
```

---

# Configure Elastic Repository

Create repository file:

```bash
vim /etc/yum.repos.d/elasticsearch.repo
```

Add:

```ini
[elasticsearch]
name=Elasticsearch repository for 9.x packages
baseurl=https://artifacts.elastic.co/packages/9.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
type=rpm-md
```

Purpose:

```text
Provides Elasticsearch packages
Provides Kibana packages
Provides Logstash packages
Provides Filebeat packages
```

---

# Install ELK Components

Install:

```bash
dnf install elasticsearch kibana logstash nginx -y
```

Components installed:

| Component | Purpose |
|------------|----------|
| Elasticsearch | Log storage |
| Kibana | Visualization |
| Logstash | Log processing |
| Nginx | Reverse proxy |

---

# Start Elasticsearch

```bash
systemctl start elasticsearch
```

Enable on boot:

```bash
systemctl enable elasticsearch
```

Verify:

```bash
systemctl status elasticsearch
```

Default Port:

```text
9200
```

---

# Start Kibana

```bash
systemctl start kibana
```

Enable:

```bash
systemctl enable kibana
```

Verify:

```bash
systemctl status kibana
```

Default Port:

```text
5601
```

---

# Configure Nginx Reverse Proxy

Instead of exposing Kibana directly:

```text
SERVER-IP:5601
```

Use:

```text
SERVER-IP:80
```

through Nginx.

Remove existing configuration:

```bash
rm -rf /etc/nginx/nginx.conf
```

Create new configuration.

---

# Nginx Reverse Proxy Architecture

```text
Browser
    │
    ▼
Nginx (Port 80)
    │
    ▼
Kibana (Port 5601)
```

Benefits:

- Cleaner URL
- SSL support
- Access control
- Load balancing capabilities

---

# Generate Kibana Enrollment Token

Elasticsearch and Kibana must be paired.

Generate token:

```bash
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

Example:

```text
eyJ2ZXIiOiI4LjEwLjAi...
```

Copy this token.

---

# Obtain Verification Code

Generate verification code:

```bash
/usr/share/kibana/bin/kibana-verification-code
```

Use this during Kibana setup.

---

# Reset Elasticsearch Admin Password

Reset password:

```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

Example:

```text
New value:
Abc123XYZ!
```

Store securely.

---

# Filebeat Installation

Create a dedicated EC2 instance for Filebeat.

Benefits:

- Log shipping separation
- Better scalability
- Easier management

---

# Configure Elastic Repository

Create:

```bash
vim /etc/yum.repos.d/elasticsearch.repo
```

Add:

```ini
[elasticsearch]
name=Elasticsearch repository for 9.x packages
baseurl=https://artifacts.elastic.co/packages/9.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
type=rpm-md
```

---

# Install Filebeat

```bash
dnf install filebeat -y
```

Verify:

```bash
systemctl status filebeat
```

---

# Install Nginx for Log Generation

Install:

```bash
dnf install nginx -y
```

Start:

```bash
systemctl start nginx
```

Generate traffic:

```bash
curl localhost
```

This creates:

```text
access.log
error.log
```

---

# Configure Filebeat

Edit:

```bash
vim /etc/filebeat/filebeat.yml
```

Configure:

```text
Enable Nginx Access Logs
Enable Nginx Error Logs
Output Elasticsearch
Protocol HTTPS
SSL Verification Disabled
```

Flow:

```text
Nginx Logs
      ↓
Filebeat
      ↓
Elasticsearch
```

---

# Verify Logs in Kibana

Open:

```text
Stack Management
```

Navigate:

```text
Index Management
```

Expected indices:

```text
filebeat-*
```

Create Data View:

```text
filebeat-*
```

Then open:

```text
Discover
```

View incoming logs.

---

# Logstash Setup

Start service:

```bash
systemctl start logstash
```

Enable:

```bash
systemctl enable logstash
```

Verify:

```bash
systemctl status logstash
```

Default Port:

```text
5044
```

---

# Logstash Configuration

Create:

```bash
vim /etc/logstash/conf.d/nginx-access-log.conf
```

Purpose:

- Receive logs from Filebeat
- Parse Nginx logs
- Convert to structured data
- Send to Elasticsearch

Flow:

```text
Filebeat
     ↓
Logstash
     ↓
Grok Parsing
     ↓
Elasticsearch
```

Benefits:

- Structured fields
- Better searching
- Better dashboards

---

# Configure Filebeat to Use Logstash

Change output:

From:

```text
Elasticsearch
```

To:

```text
Logstash
```

Flow becomes:

```text
Filebeat
     ↓
Logstash
     ↓
Elasticsearch
```

---

# Structured Logging

Without Logstash:

```text
Raw Log
```

Example:

```text
192.168.1.10 GET /index.html
```

With Logstash:

```json
{
  "client_ip": "192.168.1.10",
  "method": "GET",
  "url": "/index.html"
}
```

Benefits:

- Faster searches
- Better visualizations
- Alerting support

---

# Kubernetes (EKS) Log Collection

Install Elastic Helm Repository:

```bash
helm repo add elastic https://helm.elastic.co
```

Update:

```bash
helm repo update
```

---

# Deploy Filebeat as DaemonSet

Install:

```bash
helm install filebeat elastic/filebeat -f filebeat.yml
```

Architecture:

```text
Node 1 → Filebeat
Node 2 → Filebeat
Node 3 → Filebeat
```

Each node collects container logs.

---

# Kubernetes Log Flow

```text
Pods
  │
  ▼
Container Logs
  │
  ▼
Filebeat DaemonSet
  │
  ▼
Logstash
  │
  ▼
Elasticsearch
  │
  ▼
Kibana
```

---

# Logstash for Kubernetes Logs

Update Logstash pipeline to:

- Accept container logs
- Parse Kubernetes metadata
- Route logs to dedicated indices

Example:

```text
k8s-containers-YYYY.MM.DD
```

Benefits:

- Namespace filtering
- Pod-level troubleshooting
- Centralized cluster logging

---

# Common ELK Use Cases

## Application Monitoring

Monitor:

- Application Errors
- Request Logs
- User Activity

---

## Kubernetes Observability

Monitor:

- Pod Logs
- Node Logs
- System Logs

---

## Security Monitoring

Track:

- Failed Logins
- Unauthorized Access
- Suspicious Traffic

---

## Infrastructure Monitoring

Analyze:

- Nginx Logs
- System Logs
- Service Logs

---

# Best Practices

✅ Use dedicated Elasticsearch storage

✅ Use Logstash for log enrichment

✅ Enable HTTPS

✅ Store credentials securely

✅ Separate indices by log type

✅ Deploy Filebeat as DaemonSet in Kubernetes

✅ Monitor Elasticsearch disk usage

❌ Avoid exposing Elasticsearch publicly

❌ Avoid storing passwords in configuration files

---

# Benefits of ELK Stack

- Centralized logging
- Faster troubleshooting
- Powerful search capabilities
- Real-time monitoring
- Scalable architecture
- Kubernetes integration
- Production-grade observability

---

# Why This Setup Is Important

This ELK Stack deployment demonstrates essential observability concepts:

- Log Collection
- Log Processing
- Log Storage
- Visualization
- Kubernetes Logging

These skills are widely used in:

- DevOps Engineering
- Site Reliability Engineering (SRE)
- Platform Engineering
- Cloud Operations
- Production Infrastructure Management