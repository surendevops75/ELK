# Filebeat DaemonSet Configuration Example

This configuration is used to deploy **Filebeat** as a DaemonSet in Kubernetes for centralized log collection.

Filebeat is a lightweight log shipper from the Elastic Stack that collects logs from containers and forwards them to downstream systems such as:

- Logstash
- Elasticsearch
- Kafka

In this setup:

```text
Kubernetes Pods
        ↓
Container Logs
        ↓
Filebeat DaemonSet
        ↓
Logstash
        ↓
Elasticsearch
        ↓
Kibana
```

The configuration collects logs from all Kubernetes containers and forwards them to a Logstash server.

---

# Configuration

```yaml
daemonset:

  # Additional environment variables
  extraEnvs: []

  # Additional secret mounts
  secretMounts: []

  filebeatConfig:

    filebeat.yml: |

      # --------------------------------------------------------
      # INPUT CONFIGURATION
      # --------------------------------------------------------

      filebeat.inputs:

        - type: filestream

          # Unique input ID
          id: k8s-containers

          enabled: true

          # Kubernetes container logs are symbolic links
          prospector.scanner.symlinks: true

          # Log location
          paths:
            - /var/log/containers/*.log

          # Parse container runtime format
          parsers:
            - container: ~

          # --------------------------------------------------------
          # KUBERNETES METADATA
          # --------------------------------------------------------

          processors:

            - add_kubernetes_metadata:

                host: ${NODE_NAME}

                matchers:

                  - logs_path:

                      logs_path: "/var/log/containers/"

      # --------------------------------------------------------
      # OUTPUT CONFIGURATION
      # --------------------------------------------------------

      output.logstash:

        hosts:
          - "34.230.25.108:5044"

      # --------------------------------------------------------
      # FILEBEAT SETTINGS
      # --------------------------------------------------------

      logging.level: info

      setup.template.enabled: false

      setup.ilm.enabled: false
```

---

# Architecture Overview

```text
Application Pods
        │
        ▼
Container Logs
(/var/log/containers)
        │
        ▼
Filebeat DaemonSet
        │
        ▼
Logstash
(Port 5044)
        │
        ▼
Elasticsearch
        │
        ▼
Kibana
```

---

# What Is Filebeat?

Filebeat is a log shipper from the Elastic Stack.

Responsibilities:

- Read log files
- Parse log data
- Enrich logs
- Forward logs

Common sources:

- Linux logs
- Application logs
- Container logs
- Kubernetes logs

---

# DaemonSet

```yaml
daemonset:
```

Filebeat is typically deployed as a Kubernetes DaemonSet.

A DaemonSet ensures:

```text
One Filebeat Pod
Per Kubernetes Node
```

Example:

```text
Node-1 → Filebeat
Node-2 → Filebeat
Node-3 → Filebeat
```

This guarantees collection of logs from every node.

---

# extraEnvs

```yaml
extraEnvs: []
```

Used to inject custom environment variables.

Current configuration:

```text
No additional environment variables
```

The Helm chart automatically injects:

```text
NODE_NAME
```

which is later used by:

```yaml
add_kubernetes_metadata
```

---

# secretMounts

```yaml
secretMounts: []
```

Used when:

- TLS certificates are needed
- API keys are required
- Secure credentials are required

Current setup:

```text
No secrets mounted
```

---

# filebeat.inputs

```yaml
filebeat.inputs:
```

Defines log sources.

Each input tells Filebeat:

```text
Where logs are located
```

and

```text
How logs should be processed
```

---

# Filestream Input

```yaml
type: filestream
```

Modern Filebeat input type.

Replaces older:

```yaml
type: log
```

Benefits:

- Better performance
- Improved reliability
- Better handling of container logs

---

# Input ID

```yaml
id: k8s-containers
```

Unique identifier for the input.

Useful for:

- Troubleshooting
- Monitoring Filebeat

---

# Enabled

```yaml
enabled: true
```

Activates the input.

Without this:

```text
Logs are not collected
```

---

# Symlink Support

```yaml
prospector.scanner.symlinks: true
```

Very important in Kubernetes.

Container logs inside:

```text
/var/log/containers
```

are symbolic links.

Example:

```text
/var/log/containers/catalogue.log
        ↓
/var/log/pods/...
```

Without this setting:

```text
Filebeat cannot read container logs
```

---

# Log Path

```yaml
paths:
  - /var/log/containers/*.log
```

Filebeat monitors all container log files.

Example:

```text
catalogue.log
cart.log
user.log
payment.log
```

Every container log is collected.

---

# Container Parser

```yaml
parsers:
  - container: ~
```

Parses Kubernetes container runtime format.

Supports:

- Docker
- containerd
- CRI-O

Converts raw container logs into structured events.

---

# Kubernetes Metadata Processor

```yaml
add_kubernetes_metadata
```

Adds Kubernetes information to each log entry.

Example metadata:

```json
{
  "namespace": "roboshop",
  "pod": "catalogue-abc123",
  "container": "catalogue",
  "node": "worker-node-1"
}
```

This greatly improves searchability in Kibana.

---

# NODE_NAME

```yaml
host: ${NODE_NAME}
```

Uses the Kubernetes node name.

Example:

```text
ip-172-31-10-15
```

Allows Filebeat to map logs to the correct node.

---

# Log Path Matcher

```yaml
logs_path: "/var/log/containers/"
```

Matches logs coming from Kubernetes container directories.

This enables metadata enrichment.

---

# Output Configuration

```yaml
output.logstash:
```

Defines where logs are sent.

Current destination:

```text
34.230.25.108:5044
```

---

# Logstash

```yaml
hosts:
  - "34.230.25.108:5044"
```

Filebeat sends logs to Logstash.

Port:

```text
5044
```

is the default Beats input port.

Flow:

```text
Filebeat
     ↓
Logstash
     ↓
Elasticsearch
```

---

# Logging Level

```yaml
logging.level: info
```

Controls Filebeat logging verbosity.

Available levels:

```text
debug
info
warning
error
```

Current setting:

```text
info
```

Recommended for production.

---

# Elasticsearch Template

```yaml
setup.template.enabled: false
```

Disables automatic Elasticsearch template creation.

Reason:

```text
Logs are sent to Logstash
not directly to Elasticsearch
```

---

# ILM (Index Lifecycle Management)

```yaml
setup.ilm.enabled: false
```

Disables automatic ILM policy creation.

Reason:

```text
Logstash usually manages indexing
```

or

```text
Elasticsearch policies are managed separately
```

---

# Log Collection Workflow

```text
Application Pod
        ↓
Container Log
        ↓
/var/log/containers
        ↓
Filebeat
        ↓
Add Kubernetes Metadata
        ↓
Logstash
        ↓
Elasticsearch
        ↓
Kibana
```

---

# Real DevOps Use Cases

## Kubernetes Centralized Logging

Collect logs from:

- Pods
- Containers
- System workloads

---

## Microservices Observability

Monitor:

```text
catalogue
cart
user
payment
shipping
```

logs from a single location.

---

## Troubleshooting

Search logs by:

- Namespace
- Pod
- Container
- Node

---

## Security Monitoring

Analyze:

- Failed logins
- Unauthorized access
- Application exceptions

---

# Best Practices

✅ Deploy Filebeat as DaemonSet

✅ Enable Kubernetes Metadata

✅ Use Filestream Input

✅ Forward Logs to Logstash

✅ Centralize Logs in Elasticsearch

✅ Visualize Logs in Kibana

---

# Benefits of This Configuration

- Centralized log collection
- Automatic Kubernetes metadata enrichment
- Scalable logging architecture
- Easy troubleshooting
- Production-ready observability

---

# Why This Configuration Is Important

This configuration demonstrates production-grade log collection using:

- Filebeat
- Logstash
- Elasticsearch
- Kibana
- Kubernetes

These components form the foundation of the **ELK Stack** and are widely used in:

- DevOps Engineering
- Site Reliability Engineering (SRE)
- Cloud Operations
- Kubernetes Platforms
- Enterprise Observability Solutions