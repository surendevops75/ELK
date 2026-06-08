# Logstash Multi-Pipeline Configuration Example

This Logstash pipeline is designed to process multiple types of logs and store them in separate Elasticsearch indices.

The pipeline can handle:

- Nginx Access Logs
- Nginx Error Logs
- Kubernetes / EKS Container Logs
- Unknown or Unstructured Logs

This approach helps organize logs efficiently and improves searching, visualization, and troubleshooting in Kibana.

---

# Architecture Overview

```text
Nginx Access Logs
        │
        ▼
     Logstash
        │
        ├── nginx-access index
        │
        ├── nginx-error index
        │
        ├── k8s-containers index
        │
        └── beats-raw index
        ▼
   Elasticsearch
        ▼
      Kibana
```

---

# Logstash Configuration

```conf
input {
  beats {
    port => 5044
  }
}

filter {

  # --------------------------------------------------
  # NGINX ACCESS LOGS
  # --------------------------------------------------

  if [log][file][path] == "/var/log/nginx/access.log" or "nginx.access" in [fileset][name] {

    grok {
      match => {
        "message" => [
          '%{IPORHOST:client_ip} %{DATA:ident} %{DATA:auth} \[%{HTTPDATE:nginx_time}\] "%{WORD:http_method} %{DATA:url_path} HTTP/%{NUMBER:http_version}" %{NUMBER:status:int} (?:%{NUMBER:bytes:int}|-) "%{DATA:referrer}" "%{DATA:user_agent}" "%{DATA:extra}"'
        ]
      }

      remove_field => ["message"]
    }

    date {
      match => ["nginx_time", "dd/MMM/yyyy:HH:mm:ss Z"]
      target => "@timestamp"
      remove_field => ["nginx_time"]
    }

    mutate {
      add_field => {
        "log_type" => "nginx_access"
      }
    }
  }

  # --------------------------------------------------
  # NGINX ERROR LOGS
  # --------------------------------------------------

  else if [log][file][path] == "/var/log/nginx/error.log" {

    mutate {
      add_field => {
        "log_type" => "nginx_error"
      }
    }
  }

  # --------------------------------------------------
  # KUBERNETES CONTAINER LOGS
  # --------------------------------------------------

  else if [kubernetes] or [kubernetes][pod] {

    mutate {
      add_field => {
        "log_type" => "k8s_container"
      }
    }
  }
}

output {

  if [log_type] == "k8s_container" {

    elasticsearch {
      hosts => ["https://localhost:9200"]
      index => "k8s-containers-%{+YYYY.MM.dd}"
    }

  } else if [log_type] == "nginx_access" {

    elasticsearch {
      hosts => ["https://localhost:9200"]
      index => "nginx-access-%{+YYYY.MM.dd}"
    }

  } else if [log_type] == "nginx_error" {

    elasticsearch {
      hosts => ["https://localhost:9200"]
      index => "nginx-error-%{+YYYY.MM.dd}"
    }

  } else {

    elasticsearch {
      hosts => ["https://localhost:9200"]
      index => "beats-raw-%{+YYYY.MM.dd}"
    }
  }

  stdout {
    codec => rubydebug
  }
}
```

---

# Input Section

## Beats Input

```conf
beats {
  port => 5044
}
```

Receives logs from:

```text
Filebeat
Metricbeat
Winlogbeat
Auditbeat
```

through:

```text
Port 5044
```

Flow:

```text
Filebeat
    ↓
Port 5044
    ↓
Logstash
```

---

# Filter Section

```conf
filter
```

Processes incoming logs and enriches them before indexing.

---

# Nginx Access Log Processing

Logs matching:

```text
/var/log/nginx/access.log
```

or

```text
nginx.access
```

are treated as Nginx access logs.

---

## Grok Parsing

```conf
grok
```

Extracts structured data.

Example Log:

```text
192.168.1.10 - - [08/Jun/2026:10:15:22 +0000] "GET /login HTTP/1.1" 200 1024 "-" "Mozilla/5.0"
```

Extracted Fields:

```text
client_ip
http_method
url_path
status
bytes
user_agent
referrer
```

---

## Timestamp Conversion

```conf
date
```

Converts:

```text
08/Jun/2026:10:15:22 +0000
```

into:

```text
@timestamp
```

Benefits:

- Time-based searches
- Kibana dashboards
- Alerting

---

## Log Classification

```conf
log_type = nginx_access
```

Adds metadata.

Example:

```json
{
  "log_type": "nginx_access"
}
```

Used later for routing.

---

# Nginx Error Logs

Logs matching:

```text
/var/log/nginx/error.log
```

are tagged as:

```text
nginx_error
```

Current behavior:

```text
Store raw logs
```

Future enhancements:

- Parse error level
- Extract request IDs
- Extract upstream errors

---

# Kubernetes / EKS Logs

Condition:

```conf
[kubernetes]
```

or

```conf
[kubernetes][pod]
```

Detects logs coming from Kubernetes.

Examples:

```text
catalogue pod
cart pod
user pod
payment pod
```

Adds:

```json
{
  "log_type": "k8s_container"
}
```

Current behavior:

```text
Store raw container logs
```

Future enhancements:

- JSON parsing
- Application log parsing
- Trace ID extraction

---

# Output Section

```conf
output
```

Routes logs into different Elasticsearch indices.

---

# Kubernetes Logs Index

```conf
k8s-containers-%{+YYYY.MM.dd}
```

Example:

```text
k8s-containers-2026.06.08
```

Stores:

- Pod logs
- Container logs
- Application logs

---

# Nginx Access Logs Index

```conf
nginx-access-%{+YYYY.MM.dd}
```

Example:

```text
nginx-access-2026.06.08
```

Stores:

- HTTP Requests
- Response Codes
- URLs
- Client IPs

---

# Nginx Error Logs Index

```conf
nginx-error-%{+YYYY.MM.dd}
```

Example:

```text
nginx-error-2026.06.08
```

Stores:

- Web server errors
- Backend connectivity issues
- Application failures

---

# Fallback Index

Unknown logs are stored in:

```conf
beats-raw-%{+YYYY.MM.dd}
```

Example:

```text
beats-raw-2026.06.08
```

Useful for:

- Troubleshooting
- New log sources
- Unparsed events

---

# Elasticsearch Connection

```conf
hosts => ["https://localhost:9200"]
```

Connects to Elasticsearch.

Default Elasticsearch port:

```text
9200
```

---

# SSL Configuration

```conf
ssl_enabled => true
```

Encrypts traffic between:

```text
Logstash
     ↓
Elasticsearch
```

---

## SSL Verification

```conf
ssl_verification_mode => "none"
```

Useful for:

```text
Development
Testing
Lab Environments
```

Production recommendation:

```text
Use valid certificates
Enable certificate verification
```

---

# Debug Output

```conf
stdout {
  codec => rubydebug
}
```

Prints processed events to terminal.

Example:

```json
{
  "client_ip": "192.168.1.10",
  "status": 200,
  "log_type": "nginx_access"
}
```

Useful for:

- Debugging
- Pipeline testing
- Troubleshooting

---

# Complete Log Flow

```text
Nginx Access Logs
        │
        ▼
      Grok
        │
        ▼
 nginx-access Index

Nginx Error Logs
        │
        ▼
 nginx-error Index

Kubernetes Logs
        │
        ▼
k8s-containers Index

Unknown Logs
        │
        ▼
 beats-raw Index
```

---

# Real DevOps Use Cases

## Kubernetes Logging

Collect logs from:

```text
catalogue
cart
user
payment
shipping
frontend
```

pods.

---

## Nginx Monitoring

Track:

- Traffic
- Response Codes
- User Agents
- URLs

---

## Security Analysis

Analyze:

- Suspicious IPs
- Failed Requests
- Attack Patterns

---

## Centralized Observability

Store all infrastructure logs in Elasticsearch.

---

# Best Practices

✅ Separate logs into dedicated indices

✅ Parse access logs using Grok

✅ Convert timestamps

✅ Use Kubernetes metadata

✅ Use daily indices

✅ Enable TLS

✅ Keep fallback index for unknown logs

❌ Avoid storing passwords directly in pipeline files

❌ Disable SSL verification in production

---

# Benefits of This Architecture

- Centralized logging
- Better search performance
- Easier Kibana dashboards
- Improved troubleshooting
- Scalable observability platform
- Production-ready ELK architecture

---

# Why This Configuration Is Important

This pipeline demonstrates advanced ELK Stack concepts:

- Multi-source log ingestion
- Conditional log routing
- Nginx log parsing
- Kubernetes log collection
- Elasticsearch indexing strategies

These concepts are widely used in:

- DevOps Engineering
- Site Reliability Engineering (SRE)
- Platform Engineering
- Kubernetes Operations
- Production Observability Platforms