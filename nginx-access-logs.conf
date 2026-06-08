# Logstash Pipeline Configuration Example

This Logstash pipeline receives logs from Filebeat, parses Nginx access logs, enriches them with additional metadata, and stores them in Elasticsearch.

The pipeline performs:

- Receives logs from Filebeat
- Parses Nginx access logs using Grok
- Extracts useful fields
- Converts timestamps
- Converts data types
- Parses User-Agent information
- Sends structured logs to Elasticsearch
- Outputs logs to console for debugging

This architecture is commonly used in ELK Stack deployments.

---

# Architecture Overview

```text
Nginx
   ↓
Filebeat
   ↓
Logstash
   ↓
Grok Parsing
   ↓
Data Enrichment
   ↓
Elasticsearch
   ↓
Kibana
```

---

# Logstash Configuration

```conf
# --------------------------------------------------------
# INPUT SECTION
# --------------------------------------------------------

input {

  beats {

    # Filebeat sends logs to this port
    port => 5044
  }
}

# --------------------------------------------------------
# FILTER SECTION
# --------------------------------------------------------

filter {

  grok {

    match => {
      "message" => '%{IPORHOST:client_ip} %{DATA:ident} %{DATA:auth} \[%{HTTPDATE:nginx_time}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status} %{NUMBER:bytes} "%{DATA:referrer}" "%{DATA:user_agent}" "%{DATA:x_forwarded_for}"'
    }

    tag_on_failure => ["_grok_nginx_access_fail"]
  }

  if "_grok_nginx_access_fail" not in [tags] {

    date {

      match => ["nginx_time", "dd/MMM/yyyy:HH:mm:ss Z"]

      target => "@timestamp"

      remove_field => ["nginx_time"]
    }

    mutate {

      convert => {

        "status" => "integer"

        "bytes" => "integer"
      }
    }

    useragent {

      source => "user_agent"

      target => "ua"
    }

    mutate {

      gsub => [

        "referrer", "^-?$", "",

        "x_forwarded_for", "^-?$", "",

        "ident", "^-?$", "",

        "auth", "^-?$", ""
      ]
    }
  }
}

# --------------------------------------------------------
# OUTPUT SECTION
# --------------------------------------------------------

output {

  elasticsearch {

    hosts => ["https://localhost:9200"]

    index => "nginx-access-%{+YYYY.MM.dd}"

    user => "elastic"

    password => "YOUR_PASSWORD"

    ssl_enabled => true

    ssl_verification_mode => "none"
  }

  stdout {

    codec => rubydebug
  }
}
```

---

# Input Section

```conf
input
```

Defines where Logstash receives logs from.

---

## Beats Input

```conf
beats {
  port => 5044
}
```

Receives logs from:

```text
Filebeat
```

on:

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

Used to process, enrich, and transform incoming logs.

---

# Grok Parser

```conf
grok
```

Parses raw Nginx access logs into structured fields.

Example Nginx Log:

```text
192.168.1.10 - - [08/Jun/2026:10:15:22 +0000] "GET /index.html HTTP/1.1" 200 1024 "-" "Mozilla/5.0" "-"
```

Extracted fields:

```text
client_ip
method
request
status
bytes
user_agent
referrer
x_forwarded_for
```

---

# Grok Failure Handling

```conf
tag_on_failure => ["_grok_nginx_access_fail"]
```

If parsing fails:

```text
_grok_nginx_access_fail
```

tag is added.

This helps identify malformed logs.

---

# Conditional Processing

```conf
if "_grok_nginx_access_fail" not in [tags]
```

Only successfully parsed logs continue through the pipeline.

Failed logs skip enrichment steps.

---

# Date Filter

```conf
date
```

Converts:

```text
08/Jun/2026:10:15:22 +0000
```

into Elasticsearch timestamp format.

Before:

```text
nginx_time
```

After:

```text
@timestamp
```

Benefits:

- Accurate time-based searches
- Kibana time filtering
- Alerting support

---

# Remove Original Time Field

```conf
remove_field => ["nginx_time"]
```

Removes temporary field after conversion.

Keeps documents cleaner.

---

# Mutate Filter

```conf
mutate
```

Used to transform fields.

---

## Convert Status Code

```conf
"status" => "integer"
```

Before:

```text
"200"
```

After:

```text
200
```

---

## Convert Bytes

```conf
"bytes" => "integer"
```

Before:

```text
"1024"
```

After:

```text
1024
```

Benefits:

- Numeric aggregation
- Faster queries
- Better visualizations

---

# User Agent Parsing

```conf
useragent
```

Parses browser information.

Input:

```text
Mozilla/5.0 Chrome/138
```

Output:

```json
{
  "ua": {
    "name": "Chrome",
    "os": "Windows",
    "device": "Other"
  }
}
```

Useful for:

- Browser analytics
- Device analytics
- User behavior analysis

---

# Cleanup Invalid Values

```conf
gsub
```

Converts:

```text
-
```

into:

```text
empty value
```

Applied to:

- referrer
- x_forwarded_for
- ident
- auth

Benefits:

- Cleaner dashboards
- Better filtering
- Improved search experience

---

# Output Section

```conf
output
```

Defines where processed logs are sent.

---

# Elasticsearch Output

```conf
elasticsearch
```

Stores structured logs in Elasticsearch.

---

## Elasticsearch Host

```conf
hosts => ["https://localhost:9200"]
```

Target Elasticsearch server.

Default Elasticsearch port:

```text
9200
```

---

## Index Naming

```conf
index => "nginx-access-%{+YYYY.MM.dd}"
```

Creates daily indices.

Examples:

```text
nginx-access-2026.06.08
nginx-access-2026.06.09
nginx-access-2026.06.10
```

Benefits:

- Easier retention management
- Faster queries
- Better index organization

---

## Authentication

```conf
user => "elastic"
```

Elasticsearch username.

---

```conf
password => "YOUR_PASSWORD"
```

Elasticsearch password.

⚠️ Best Practice:

Do not hardcode passwords.

Use:

- Environment Variables
- Kubernetes Secrets
- Vault
- AWS Secrets Manager

---

# SSL Configuration

```conf
ssl_enabled => true
```

Enables encrypted communication.

Flow:

```text
Logstash
     ↓
TLS
     ↓
Elasticsearch
```

---

## SSL Verification

```conf
ssl_verification_mode => "none"
```

Disables certificate verification.

Useful for:

```text
Lab Environment
Development
Testing
```

Not recommended for production.

Production:

```text
Use valid CA certificates
```

---

# Debug Output

```conf
stdout
```

Prints processed events to terminal.

---

## Ruby Debug Codec

```conf
codec => rubydebug
```

Shows complete event structure.

Example:

```json
{
  "client_ip": "192.168.1.10",
  "method": "GET",
  "status": 200,
  "bytes": 1024
}
```

Useful for:

- Troubleshooting
- Pipeline validation
- Testing filters

---

# Complete Log Flow

```text
Nginx
   ↓
Access Log
   ↓
Filebeat
   ↓
Logstash Input (5044)
   ↓
Grok Parsing
   ↓
Date Conversion
   ↓
User Agent Parsing
   ↓
Data Cleanup
   ↓
Elasticsearch
   ↓
Kibana Dashboard
```

---

# Real DevOps Use Cases

## Web Server Monitoring

Track:

- Requests
- Status Codes
- Response Sizes
- Client IPs

---

## Security Monitoring

Identify:

- Suspicious IPs
- Brute Force Attempts
- Unauthorized Access

---

## Performance Analysis

Monitor:

- Traffic Volume
- Popular URLs
- Browser Usage
- Response Trends

---

## Centralized Logging

Collect logs from:

- Nginx
- Apache
- Kubernetes Ingress
- Application Servers

---

# Best Practices

✅ Parse logs using Grok

✅ Convert timestamps

✅ Convert numeric fields

✅ Parse User-Agent information

✅ Use daily indices

✅ Enable TLS

✅ Use structured logging

❌ Avoid hardcoded passwords

❌ Disable SSL verification in production

---

# Why This Configuration Is Important

This Logstash pipeline demonstrates core ELK Stack concepts:

- Filebeat log collection
- Logstash parsing
- Data enrichment
- Elasticsearch indexing
- Kibana visualization

These concepts are widely used in:

- DevOps Engineering
- Site Reliability Engineering (SRE)
- Observability Platforms
- Security Monitoring
- Production Logging Systems