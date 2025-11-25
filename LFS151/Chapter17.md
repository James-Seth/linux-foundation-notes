# Chapter 17: Container Debugging, Logging, and Monitoring

## Table of Contents
- [Overview](#overview)
- [Container Monitoring Challenges](#container-monitoring-challenges)
- [Native Container Tools](#native-container-tools)
- [Specialized Monitoring Tools](#specialized-monitoring-tools)
- [Study Questions](#study-questions)
- [Quick Reference](#quick-reference)
- [Glossary](#glossary)
- [Summary](#summary)

---

## Overview

Container debugging, logging, and monitoring require specialized approaches due to the ephemeral nature of containers and their distributed architecture.

**Traditional Tools:**
- `strace` - System call tracer
- `SAR` (System Activity Reporter) - System performance monitoring
- `tcpdump` - Network packet analyzer
- `GDB` (GNU Project Debugger) - Source-level debugger
- `syslog` - System logging facility
- `Nagios` - Infrastructure monitoring
- `Zabbix` - Enterprise monitoring solution

---

## Container Monitoring Challenges

**Key Challenges:**
1. **Ephemeral Nature** - Containers are temporary; logs disappear when containers are deleted
2. **No Kernel Components** - Containers share the host kernel, limiting traditional monitoring
3. **Small Footprint** - Installing debugging tools conflicts with minimal container design
4. **Scale Complexity** - Collecting data from hundreds of containers individually is impractical

**Solution Approach:**
- Use external tools for centralized monitoring
- Leverage host OS control over container processes
- Implement cluster-wide observability (metrics + logs + tracing)

```
┌─────────────────────────────────────────────────────────────┐
│                    Observability Stack                     │
├─────────────────┬─────────────────┬─────────────────────────┤
│     Metrics     │      Logs       │    Distributed Tracing │
│   (Prometheus)  │   (Fluentd)     │      (Jaeger)          │
└─────────────────┴─────────────────┴─────────────────────────┘
```

**Tool Categories:**
- **Debugging:** Docker CLI, Kubernetes CLI, Podman CLI, nerdctl CLI, crictl CLI, Sysdig
- **Logging:** Docker/Podman Logging Drivers, centralized logging solutions
- **Monitoring:** cAdvisor, Prometheus, Datadog, New Relic, Dynatrace

---

## Native Container Tools

All major container runtimes provide built-in debugging, logging, and monitoring capabilities.

### Debugging Commands

**Inspect Container Details:**
```bash
# Docker
docker inspect <container_id>

# Podman  
podman inspect <container_id>

# nerdctl (containerd CLI)
nerdctl inspect <container_id>

# crictl (CRI-compatible)
crictl inspect <container_id>
```

### Logging Commands

**View Container Logs:**
```bash
# Real-time log following
docker logs -f <container_id>
podman logs -f <container_id>
nerdctl logs -f <container_id>
crictl logs <container_id>

# Show last 100 lines with timestamps
docker logs --tail 100 -t <container_id>
```

**Docker Logging Drivers:**
- `json-file` (default) - JSON format logs
- `syslog` - System logger integration
- `journald` - systemd journal integration
- `gelf` - Graylog Extended Log Format
- `fluentd` - Fluentd integration
- `awslogs` - AWS CloudWatch Logs
- `splunk` - Splunk integration

**Podman Logging Drivers:**
- `k8s-file` (default) - Kubernetes-compatible JSON
- `journald` - systemd journal
- `none` - No logging
- `passthrough` - Pass to parent process

### Monitoring Commands

**Resource Statistics:**
```bash
# Live resource usage
docker stats
podman stats  
nerdctl stats
crictl stats

# Process information
docker top <container_id>
podman top <container_id>
```

```
Example Output:
CONTAINER ID   NAME     CPU %   MEM USAGE/LIMIT   MEM %   NET I/O       BLOCK I/O
abc123def456   webapp   0.50%   256MiB/2GiB       12.8%   1.2MB/800kB   0B/0B
```

---

## Specialized Monitoring Tools

### Sysdig
> **"strace + tcpdump + htop + iftop + lsof + awesome sauce"**

**Components:**
- **Sysdig Open Source** - System call capture and analysis
- **Sysdig Monitor** - Full-stack monitoring with dashboards (paid)
- **Sysdig Secure** - Security and compliance monitoring (paid)

**Key Features:**
- Captures low-level system information from Linux instances
- Native support for Docker, Kubernetes, Mesos, AWS, GCP
- Kubernetes-aware monitoring
- Built for production with minimal overhead
- Requires kernel module for system call capture

**Installation:**
```bash
# Install Sysdig agent on all nodes
curl -s https://download.sysdig.com/stable/install-agent | sudo bash
```

### cAdvisor (Container Advisor)

**Purpose:** Real-time container resource usage and performance monitoring

**Deployment:**
```bash
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:latest
```

**Access:** Navigate to `http://host_IP:8080` for web interface

**Features:**
- Exposes container statistics as Prometheus metrics
- REST API for programmatic access
- InfluxDB export support
- Per-container resource utilization tracking

```
┌─────────────────────────────────────────────────────┐
│                cAdvisor Dashboard                   │
├─────────────────────────────────────────────────────┤
│ Running Containers: 12                              │
│ Docker Version: 20.10.21                            │
│ Total Images: 45                                     │
├─────────────────────────────────────────────────────┤
│ CPU Usage:     [████████░░] 80%                     │
│ Memory Usage:  [████████░░] 75%                     │
│ Network I/O:   1.2 GB / 800 MB                     │
└─────────────────────────────────────────────────────┘
```

### Elasticsearch + Kibana

**Purpose:** Distributed search, analytics, and data visualization

**Core Components:**
- **Elasticsearch** - Search and analytics engine for data indexing
- **Kibana** - Interactive data visualization interface

**Key Features:**
- Real-time search for structured/unstructured data
- Distributed, scalable, highly available
- Cross-cluster and cross-datacenter replication
- Advanced security with encryption
- Supports multiple data types (text, numbers, geospatial, time series)
- Machine learning and alerting capabilities
- RESTful APIs and multiple language clients (Java, Python, SQL, PHP)

```
┌─────────────────────────────────────────────────────┐
│                Kibana Dashboard                     │
├─────────────────────────────────────────────────────┤
│ [Log Volume Chart    ] [Error Rate Trend         ] │
│ [Top Error Messages  ] [Response Time Histogram  ] │
│ [Geographic Map      ] [Service Health Matrix    ] │
└─────────────────────────────────────────────────────┘
```

### Fluentd

**Purpose:** Open source data collector for unified logging

> **"Unify data collection and consumption for better use and understanding of data"**

**Architecture:**
- Structures data as JSON
- 500+ plugins for input/output sources
- Filtering, buffering, and routing capabilities
- CNCF member project

**Docker Integration:**
```bash
# Configure Fluentd as Docker logging driver
docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 myapp
```

```
┌─────────────────────────────────────────────────────┐
│                Fluentd Data Flow                    │
├─────────────────────────────────────────────────────┤
│ [Input Sources] → [Filter/Route] → [Output Dest.]  │
│                                                     │
│ • Docker logs     • Parse          • Elasticsearch │
│ • Kubernetes      • Transform      • S3            │
│ • Apache logs     • Buffer         • MongoDB       │
│ • Syslog         • Route          • Kafka         │
└─────────────────────────────────────────────────────┘
```

### Datadog

**Purpose:** Monitoring and analytics platform as a service

**Capabilities:**
- Infrastructure, network, and application monitoring
- Integrated security monitoring
- 600+ third-party integrations (EC2, Apache, Java, MySQL, etc.)
- AI-powered threat detection

**Deployment:**
```bash
# Install Datadog agent
DD_API_KEY=your_api_key bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
```

**Kubernetes Dashboard Features:**
- Cluster nodes and resource utilization
- Running/stopped container counts  
- Resource-consuming pod identification
- Per-node container distribution

```
┌─────────────────────────────────────────────────────┐
│            Datadog Kubernetes Overview              │
├─────────────────────────────────────────────────────┤
│ Cluster: production-cluster     Nodes: 8           │
│ Running Containers: 156         Stopped: 12        │
│ ──────────────────────────────────────────────────  │
│ Top Resource Consuming Pods:                        │
│ • web-frontend-789abc (CPU: 85%)                   │
│ • database-primary-456def (Memory: 78%)            │
│ • worker-queue-123ghi (Network: 65%)               │
└─────────────────────────────────────────────────────┘
```

### Prometheus

**Purpose:** Open source monitoring and alerting toolkit (CNCF graduated project)

**Architecture Components:**
- **Prometheus Server** - Scrapes and stores metrics
- **Alertmanager** - Handles alerts and notifications  
- **Pushgateway** - For short-lived job metrics
- **Exporters** - Expose metrics from third-party systems
- **Client Libraries** - For application instrumentation

**Key Features:**
- Multi-dimensional data model with time series
- PromQL query language for data analysis
- Pull and push-based metric collection
- Service discovery and static configuration
- Integration with Grafana for dashboards

```prometheus
# Example Prometheus query
rate(http_requests_total[5m])

# Container CPU usage by pod
rate(container_cpu_usage_seconds_total[1m]) * 100
```

```
┌─────────────────────────────────────────────────────┐
│              Prometheus Architecture                │
├─────────────────────────────────────────────────────┤
│                                                     │
│ [Targets] → [Prometheus Server] → [Grafana/UI]     │
│     ↓              ↓                   ↑            │
│ [Service      [Time Series      [Alertmanager]     │
│ Discovery]     Database]                           │
│                                                     │
│ • Kubernetes   • Metrics storage   • PagerDuty     │
│ • Consul       • PromQL queries    • Slack         │
│ • Static       • HTTP API          • Email         │
└─────────────────────────────────────────────────────┘
```

### Grafana

**Purpose:** Open source analytics and interactive visualization platform

**Key Features:**
- Dashboard creation with 150+ built-in panel types
- Multi-data source support (Prometheus, InfluxDB, Elasticsearch, etc.)
- Alerting with notification channels (Slack, PagerDuty, email)
- User management and team collaboration
- Plugin ecosystem with 100+ data sources
- Templating and variable support
- Time range controls and query inspector

**Integration with Prometheus:**
```yaml
# docker-compose.yml for Grafana + Prometheus
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
  
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana-data:/var/lib/grafana

volumes:
  grafana-data:
```

**Popular Dashboards:**
- **Kubernetes Cluster Monitoring** (ID: 10000)
- **Docker Container Dashboard** (ID: 395)
- **Node Exporter Full** (ID: 1860)
- **Nginx Ingress Controller** (ID: 9614)

```
┌─────────────────────────────────────────────────────┐
│                Grafana Dashboard                    │
├─────────────────────────────────────────────────────┤
│ [CPU Usage Graph    ] [Memory Usage Gauge        ] │
│ [Network I/O Chart  ] [Disk Usage Bar Chart      ] │
│ [Container Count    ] [Alert Notifications       ] │
│ [Query Inspector    ] [Time Range: Last 1h       ] │
└─────────────────────────────────────────────────────┘
```

### Jaeger

**Purpose:** Open source distributed tracing platform (CNCF graduated project)

**Architecture Components:**
- **Jaeger Client** - Language-specific SDKs for instrumentation
- **Jaeger Agent** - Network daemon for batching and forwarding traces
- **Jaeger Collector** - Receives traces and processes them
- **Jaeger Query** - Retrieves traces from storage and hosts UI
- **Storage Backend** - Elasticsearch, Cassandra, or Kafka

**Key Features:**
- Distributed context propagation
- Distributed transaction monitoring  
- Root cause analysis for performance issues
- Service dependency analysis
- OpenTracing/OpenTelemetry compatible

**Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:latest
        ports:
        - containerPort: 16686  # UI
        - containerPort: 14268  # HTTP collector
        env:
        - name: COLLECTOR_ZIPKIN_HTTP_PORT
          value: "9411"
---
apiVersion: v1
kind: Service
metadata:
  name: jaeger-service
spec:
  selector:
    app: jaeger
  ports:
  - name: ui
    port: 16686
    targetPort: 16686
  - name: collector
    port: 14268
    targetPort: 14268
```

**Trace Example:**
```
┌─────────────────────────────────────────────────────┐
│              Distributed Trace View                 │
├─────────────────────────────────────────────────────┤
│ Service: frontend    Duration: 245ms                │
│ ├── auth-service    Duration: 45ms                  │
│ ├── user-service    Duration: 67ms                  │
│ └── order-service   Duration: 133ms                 │
│     ├── database    Duration: 89ms                  │
│     └── payment     Duration: 44ms                  │
└─────────────────────────────────────────────────────┘
```

### New Relic

**Purpose:** Full-stack observability platform for modern cloud environments

**Key Capabilities:**
- **APM (Application Performance Monitoring)** - Code-level insights
- **Infrastructure Monitoring** - Server, container, and cloud metrics
- **Browser Monitoring** - Real user monitoring (RUM)
- **Synthetics** - Proactive monitoring with simulated users
- **Mobile Monitoring** - iOS/Android app performance
- **Logs in Context** - Correlated log data with traces
- **AI/ML Insights** - Anomaly detection and root cause analysis

**Container Integration:**
```bash
# Install New Relic Infrastructure agent
docker run \
  --name newrelic-infra \
  --network=host \
  --cap-add=SYS_PTRACE \
  --privileged \
  --pid=host \
  --volume=/:/host:ro \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  --env NRIA_LICENSE_KEY=your_license_key \
  newrelic/infrastructure:latest
```

**Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: newrelic-infra
  namespace: newrelic
spec:
  selector:
    matchLabels:
      name: newrelic-infra
  template:
    metadata:
      labels:
        name: newrelic-infra
    spec:
      serviceAccount: newrelic
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: newrelic-infra
        image: newrelic/infrastructure-k8s:latest
        env:
        - name: NRIA_LICENSE_KEY
          valueFrom:
            secretKeyRef:
              name: newrelic-license
              key: license
        - name: NRIA_DISPLAY_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: spec.nodeName
        volumeMounts:
        - name: host-volume
          mountPath: /host
          readOnly: true
        - name: dev-dir
          mountPath: /dev
          readOnly: true
        - name: proc-dir
          mountPath: /host/proc
          readOnly: true
        - name: sys-dir
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: host-volume
        hostPath:
          path: /
      - name: dev-dir
        hostPath:
          path: /dev
      - name: proc-dir
        hostPath:
          path: /proc
      - name: sys-dir
        hostPath:
          path: /sys
```

### Splunk

**Purpose:** Enterprise data platform for real-time insights

**Deployment Options:**
- On-premises installation
- Public cloud deployment  
- Hybrid cloud architecture

**Key Features:**
- Fast real-time data streaming and analysis
- Interactive dashboards with AR/VR support
- Advanced access control and clustering
- DevSecOps automation and orchestration
- Custom investigative methods

### OpenTelemetry  

**Purpose:** Vendor-agnostic observability framework (CNCF incubating project)

**Core Component:** 
- **Collector** - Single agent supporting multiple telemetry formats (Jaeger, Prometheus, Fluent Bit)

**Supported Languages:** .NET, C++, Java, JavaScript, Go, Python, PHP, Ruby, Rust

**Data Types:**
- Metrics - Performance measurements
- Logs - Event records  
- Traces - Request flow tracking

```yaml
# OpenTelemetry Collector configuration
receivers:
  prometheus:
    config:
      scrape_configs:
        - job_name: 'kubernetes-pods'
          
exporters:
  jaeger:
    endpoint: jaeger-collector:14250
  prometheus:
    endpoint: "prometheus:8889"
```

### Dynatrace

**Purpose:** AI-powered unified observability platform

**Key Integrations:**
- **AWS** - EC2, ECS, Lambda, EKS monitoring with Keptn
- **Azure** - App Service, AKS, Functions auto-discovery
- **GCP** - GCE, GKE, Anthos monitoring
- **Kubernetes** - OneAgent Operator for cluster-wide observability
- **OpenShift** - CRI-O and Docker monitoring at scale

**OneAgent Features:**
- Automatic deployment to cluster nodes
- Real-time discovery and monitoring
- Infrastructure to microservice visibility
- GitOps automation with Keptn integration

---

## Common Troubleshooting Scenarios

### Container Performance Issues

**Scenario 1: High Memory Usage (OOMKilled)**
```bash
# Identify OOMKilled containers
kubectl get pods --field-selector=status.phase=Failed
kubectl describe pod <pod-name> | grep -i "oomkilled\|memory"

# Check resource limits and requests
kubectl describe pod <pod-name> | grep -A 10 -i "limits\|requests"

# Monitor memory usage in real-time
kubectl top pod <pod-name> --containers
docker stats <container-id>
```

**Solution Steps:**
1. Increase memory limits in deployment specs
2. Optimize application memory usage
3. Implement memory profiling
4. Consider horizontal pod autoscaling

**Scenario 2: High CPU Usage**
```bash
# Identify CPU-intensive containers
kubectl top pods --sort-by=cpu
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" | sort -k2 -nr

# Profile CPU usage with Sysdig
sudo sysdig -pc -c topprocs_cpu

# Check CPU throttling
cat /sys/fs/cgroup/cpu/docker/<container-id>/cpu.stat | grep throttled
```

### Network Connectivity Issues

**Scenario 3: Service Discovery Problems**
```bash
# Test DNS resolution
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl exec -it <pod-name> -- dig <service-name>.default.svc.cluster.local

# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe service <service-name>

# Test connectivity between pods
kubectl exec -it <source-pod> -- telnet <target-service> <port>
kubectl exec -it <source-pod> -- wget -qO- <target-service>:<port>/health
```

**Scenario 4: Port Binding Issues**
```bash
# Check port usage
netstat -tulpn | grep <port>
ss -tulpn | grep <port>

# Verify container port mappings
docker port <container-name>
kubectl describe pod <pod-name> | grep -A 5 -i ports

# Test external connectivity
curl -v http://localhost:<port>/health
telnet <host-ip> <port>
```

### Log Analysis Issues

**Scenario 5: Missing or Truncated Logs**
```bash
# Check logging driver configuration
docker info | grep -i "logging driver"

# Verify log retention settings
docker system df
docker system prune --volumes

# Check journald logs for Podman
sudo journalctl -u <container-service> --since "1 hour ago"

# Elasticsearch log ingestion issues
curl -X GET "elasticsearch:9200/_cluster/health?pretty"
curl -X GET "elasticsearch:9200/_cat/indices?v"
```

**Scenario 6: Log Parsing Failures**
```bash
# Test Fluentd configuration
fluentd --dry-run -c /etc/fluent/fluent.conf

# Check Fluentd buffer status
curl http://localhost:24220/api/plugins.json

# Debug log format issues
kubectl logs <fluentd-pod> -c fluentd | grep -i error
```

### Monitoring Tool Issues

**Scenario 7: Prometheus Target Discovery**
```bash
# Check Prometheus targets
curl http://prometheus:9090/api/v1/targets

# Verify service discovery
kubectl get servicemonitors
kubectl describe servicemonitor <monitor-name>

# Check metrics endpoint accessibility
curl http://<target-service>:<port>/metrics
kubectl exec -it <prometheus-pod> -- wget -qO- <target>:port/metrics
```

**Scenario 8: Grafana Dashboard Issues**
```bash
# Test Grafana data source connectivity
curl -X POST http://admin:password@grafana:3000/api/datasources/proxy/1/api/v1/query?query=up

# Check dashboard provisioning
kubectl logs <grafana-pod> | grep -i "dashboard\|provisioning"

# Verify data source configuration
curl -X GET http://admin:password@grafana:3000/api/datasources
```

---

## Tool Comparison and Selection Guide

### Monitoring Tools Comparison Matrix

| **Criteria** | **Prometheus + Grafana** | **Datadog** | **New Relic** | **Dynatrace** | **Sysdig** | **ELK Stack** |
|--------------|---------------------------|-------------|---------------|---------------|------------|---------------|
| **Cost** | Free (Open Source) | $15-23/host/month | $25-349/host/month | $69-579/host/month | $20-35/host/month | Free (Open Source) |
| **Setup Complexity** | Medium-High | Low | Low | Low | Medium | High |
| **Kubernetes Native** | Excellent | Good | Good | Excellent | Excellent | Good |
| **Alerting** | Excellent | Excellent | Excellent | Excellent | Good | Good |
| **Distributed Tracing** | Manual (Jaeger) | Built-in | Built-in | Built-in | Basic | Manual |
| **APM Features** | Basic | Excellent | Excellent | Excellent | Good | Basic |
| **Log Management** | Manual (Loki) | Built-in | Built-in | Built-in | Basic | Excellent |
| **Custom Metrics** | Excellent | Good | Good | Limited | Good | Good |
| **Community** | Large | Medium | Medium | Small | Medium | Large |
| **Learning Curve** | Steep | Gentle | Gentle | Gentle | Medium | Steep |

### Selection Decision Tree

```
┌─────────────────────────────────────────────────────┐
│              Monitoring Tool Selection               │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Budget Available? ┌─ Yes ─┐ Large Team? ┌─ Yes ─┐  │
│                   │       │             │       │  │
│                   │       └─ No ─┐      │  ┌─ Dynatrace  │
│                   │              │      │  │    (Enterprise)  │
│                   │         ┌─ Datadog   │  │             │
│                   │         │ (Mid-size)  │  └─ New Relic │
│                   │         │             │    (Growth)   │
│                   │         └─────────────┘              │
│         ┌─ No ─┐  │                                       │
│         │      │  │                                       │
│    ┌─ Prometheus + │                                      │
│    │   Grafana     │                                      │
│    │   (DIY)       │                                      │
│    │               │                                      │
│    └─ Open Source  │                                      │
│       ELK Stack    │                                      │
│                    │                                      │
└─────────────────────────────────────────────────────────┘
```

### Use Case Recommendations

**Development/Testing Environments:**
- **Best Choice:** Prometheus + Grafana + Jaeger
- **Reason:** Free, comprehensive, learning-friendly
- **Setup Time:** 2-4 hours

**Small to Medium Production (< 100 nodes):**
- **Best Choice:** Datadog or New Relic
- **Reason:** Quick setup, managed service, good support
- **Setup Time:** 1-2 hours

**Large Enterprise Production (> 100 nodes):**
- **Best Choice:** Dynatrace or Datadog Enterprise
- **Reason:** Advanced AI, enterprise features, dedicated support
- **Setup Time:** 4-8 hours (with professional services)

**Kubernetes-First Organizations:**
- **Best Choice:** Prometheus + Grafana + Jaeger
- **Reason:** Native integration, CNCF ecosystem alignment
- **Setup Time:** 4-6 hours (with Helm charts)

**Compliance/Security-Critical:**
- **Best Choice:** Sysdig Secure + Prometheus
- **Reason:** Deep system visibility, security monitoring
- **Setup Time:** 6-8 hours

---

## Study Questions

1. **What are the main challenges of monitoring containerized applications?**
2. **Which command would you use to view real-time logs from a Docker container?**
3. **What is the default logging driver for Docker and Podman?**
4. **How does Sysdig differ from traditional monitoring tools?**
5. **What port does cAdvisor use by default for its web interface?**
6. **Name three data types that OpenTelemetry can handle.**
7. **What is the core component of OpenTelemetry architecture?**
8. **Which CNCF project status does Prometheus have?**
9. **How does Grafana complement Prometheus in a monitoring stack?**
10. **What are the four main components of Jaeger's distributed tracing architecture?**
11. **Which New Relic capability provides real user monitoring for web applications?**
12. **What are the common causes of OOMKilled container failures?**
13. **How would you troubleshoot service discovery issues in Kubernetes?**
14. **What factors should influence your choice between commercial and open-source monitoring tools?**
15. **Which monitoring solution would you recommend for a small startup vs. large enterprise?**

---

## Quick Reference

| Tool | Type | Key Feature | CNCF Status |
|------|------|-------------|-------------|
| **Sysdig** | System monitoring | System call capture | - |
| **cAdvisor** | Container monitoring | Prometheus metrics | Google project |
| **Elasticsearch** | Search & analytics | Real-time search | - |
| **Fluentd** | Log collection | JSON structure | Member |
| **Prometheus** | Monitoring | Time series DB | Graduated |
| **Grafana** | Visualization | Dashboard platform | - |
| **Jaeger** | Distributed tracing | OpenTracing compatible | Graduated |
| **Datadog** | APM platform | 600+ integrations | - |
| **New Relic** | APM platform | Full-stack observability | - |
| **OpenTelemetry** | Observability | Vendor-agnostic | Incubating |
| **Dynatrace** | AI platform | OneAgent | - |

**Common Commands:**
```bash
# Container inspection
docker/podman inspect <container_id>

# Live logs
docker/podman logs -f <container_id>

# Resource stats
docker/podman stats

# Process info
docker/podman top <container_id>
```

**Complete Monitoring Stack (Docker Compose):**
```yaml
# docker-compose-monitoring.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14268:14268"
    environment:
      - COLLECTOR_OTLP_ENABLED=true

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    privileged: true
    devices:
      - /dev/kmsg:/dev/kmsg

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  fluentd:
    image: fluent/fluentd:edge-debian
    ports:
      - "24224:24224"
      - "24224:24224/udp"
    volumes:
      - ./fluentd/fluent.conf:/fluentd/etc/fluent.conf
    depends_on:
      - elasticsearch

volumes:
  prometheus-data:
  grafana-data:
  elasticsearch-data:
```

**Kubernetes Monitoring Setup:**
```yaml
# monitoring-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
---
# prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: storage
          mountPath: /prometheus
        args:
        - '--config.file=/etc/prometheus/prometheus.yml'
        - '--storage.tsdb.path=/prometheus'
        - '--storage.tsdb.retention.time=15d'
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
  type: NodePort
```

---

## Glossary

> **📊 Key Terms and Definitions**
> 
> **Alertmanager:** Prometheus component for handling alerts and notifications.  
> **APM:** Application Performance Monitoring - code-level insights and performance analysis.  
> **cAdvisor:** Container Advisor - Google's tool for container resource monitoring.  
> **CNCF:** Cloud Native Computing Foundation - hosts cloud-native projects.  
> **Collector:** OpenTelemetry component that gathers and processes telemetry data.  
> **Container Runtime:** Software responsible for running containers (Docker, containerd, CRI-O).  
> **CPU Throttling:** Limiting container CPU usage when exceeding allocated resources.  
> **Dashboard:** Visual representation of metrics and data for monitoring and analysis.  
> **DevSecOps:** Integration of security practices into DevOps workflows.  
> **Distributed Tracing:** Tracking requests across multiple microservices and components.  
> **Elasticsearch:** Search and analytics engine for full-text search and data indexing.  
> **Ephemeral:** Temporary nature of containers that are frequently created and destroyed.  
> **Exporter:** Component that exposes metrics from third-party systems to Prometheus.  
> **Fluentd:** Open source data collector for unified logging layer.  
> **GDB:** GNU Project Debugger - source-level debugging tool for applications.  
> **GitOps:** Operations methodology using Git as single source of truth for infrastructure.  
> **Grafana:** Open source platform for monitoring and observability dashboards.  
> **Jaeger:** Open source distributed tracing platform for monitoring microservices.  
> **JSON:** JavaScript Object Notation - lightweight data-interchange format.  
> **Kibana:** Data visualization dashboard for Elasticsearch.  
> **Kubernetes:** Container orchestration platform for automating deployment and management.  
> **Log Aggregation:** Process of collecting logs from multiple sources into a centralized location.  
> **Metrics:** Quantitative measurements of system performance and behavior.  
> **Microservices:** Architectural approach using small, independent services.  
> **Nagios:** Traditional infrastructure monitoring system for servers and services.  
> **Namespace:** Kubernetes mechanism for isolating groups of resources.  
> **Observability:** System-wide visibility through metrics, logs, and traces.  
> **OneAgent:** Dynatrace's automatic deployment agent for full-stack monitoring.  
> **OOMKilled:** Out Of Memory termination when containers exceed memory limits.  
> **OpenTelemetry:** Vendor-neutral observability framework for collecting telemetry data.  
> **Pod:** Smallest deployable unit in Kubernetes containing one or more containers.  
> **PromQL:** Prometheus Query Language for analyzing time series data.  
> **Pushgateway:** Prometheus component for collecting metrics from short-lived jobs.  
> **REST API:** Representational State Transfer - architectural style for web services.  
> **RUM:** Real User Monitoring - measuring actual user experience and performance.  
> **SAR:** System Activity Reporter - traditional Unix/Linux performance monitoring tool.  
> **Service Discovery:** Automatic detection and registration of services in a network.  
> **Service Mesh:** Infrastructure layer for handling service-to-service communication.  
> **SLI:** Service Level Indicator - specific metric measuring service performance.  
> **SLO:** Service Level Objective - target value for service level indicators.  
> **Span:** Individual operation within a trace representing a unit of work.  
> **strace:** System call tracer for debugging and monitoring program execution.  
> **Sysdig:** Platform for container and Kubernetes security and monitoring.  
> **syslog:** Traditional system logging facility for Unix/Linux systems.  
> **tcpdump:** Network packet analyzer for troubleshooting network issues.  
> **Time Series:** Data points indexed in chronological order for trend analysis.  
> **Trace:** Record of a request's journey through distributed system components.  
> **YAML:** YAML Ain't Markup Language - human-readable data serialization standard.  
> **Zabbix:** Enterprise-class monitoring solution for networks, servers, and applications.

---

## Summary

- Container monitoring requires specialized tools due to ephemeral nature and scale
- Native container CLIs provide basic debugging, logging, and monitoring capabilities
- Centralized logging solutions (Fluentd, Elasticsearch) handle log aggregation at scale
- Prometheus + Grafana ecosystem dominates open source monitoring with CNCF backing
- Jaeger provides distributed tracing to complete the observability trinity (metrics, logs, traces)
- Commercial platforms (Datadog, New Relic, Dynatrace) offer comprehensive observability with AI features
- OpenTelemetry provides vendor-neutral observability standardization
- Tool selection depends on budget, team size, complexity requirements, and compliance needs
- Common troubleshooting scenarios include OOMKilled containers, network connectivity issues, and monitoring tool configuration problems
- Complete monitoring stack requires careful integration of multiple components for production readiness
- Success depends on proper tool selection, configuration, and team training for the chosen solution