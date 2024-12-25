**Infrastructure Requirements for the Observability Stack**

In the previous article, we discussed why our observability stack is better than commercial alternatives. Now, let’s take a closer look at the infrastructure needed to deploy this stack effectively. Whether you’re setting up for a small application or a large-scale enterprise system, this guide will help you design a scalable, reliable, and efficient infrastructure.

---

### **Core Components of the Observability Stack**
To recap, our observability stack consists of the following tools:
- **Prometheus**: Metrics collection.
- **Loki**: Log aggregation.
- **Grafana**: Visualization and dashboards.
- **Jaeger/Tempo**: Distributed tracing.
- **Alertmanager**: Centralized alert routing.
- **OpenTelemetry Collector**: Centralized telemetry data ingestion.

Each of these components requires specific infrastructure to function optimally. Let’s dive into the details.

---

### **1. Prometheus: Metrics Collector**
Prometheus is responsible for scraping and storing metrics from applications and systems.

#### **Requirements**:
1. **Hardware**:
   - **Small Deployment**: 2 CPUs, 8 GB RAM, 50 GB SSD.
   - **Large Deployment**: 4 CPUs, 16 GB RAM, 100 GB SSD (or more for high data retention).

2. **Storage**:
   - Prometheus stores time-series data locally.
   - For long-term storage and scalability, integrate with **Thanos** or **Cortex** to use object storage (e.g., S3, MinIO).

3. **Redundancy**:
   - Run two Prometheus instances in active-passive mode for failover.

4. **Networking**:
   - Open ports for Prometheus to scrape data from exporters (default: `9090`).

#### **Scaling**:
- Horizontal scaling can be achieved with **Prometheus federation** or by adding Thanos for distributed metrics storage.

---

### **2. Loki: Log Aggregation**
Loki stores logs in a lightweight, label-based format, making it efficient and scalable.

#### **Requirements**:
1. **Hardware**:
   - **Small Deployment**: 2 CPUs, 8 GB RAM, 50 GB SSD.
   - **Large Deployment**: 4 CPUs, 16 GB RAM, 200 GB SSD or more for high log volumes.

2. **Storage**:
   - Use S3-compatible object storage for long-term log retention.
   - Store recent logs on local disks for faster querying.

3. **Redundancy**:
   - Deploy Loki in a distributed setup with multiple instances for redundancy.

4. **Networking**:
   - Open ports for log ingestion (default: `3100`) and configure log forwarders (e.g., **Promtail**, **Fluentd**).

#### **Scaling**:
- Add more Loki nodes to handle higher log ingestion rates.

---

### **3. Grafana: Visualization Layer**
Grafana ties all components together, providing a unified view of metrics, logs, and traces.

#### **Requirements**:
1. **Hardware**:
   - **Small Deployment**: 2 CPUs, 4 GB RAM, 10 GB SSD.
   - **Large Deployment**: 4 CPUs, 8 GB RAM, 50 GB SSD for heavy concurrent users.

2. **Redundancy**:
   - Use a load balancer (e.g., HAProxy, Nginx) to distribute traffic across multiple Grafana instances.

3. **Networking**:
   - Open port `3000` for web access to the Grafana dashboard.

#### **Scaling**:
- Scale horizontally by adding Grafana instances for high user loads.

---

### **4. Jaeger/Tempo: Distributed Tracing**
Jaeger or Tempo captures and stores distributed traces, helping to analyze request flows across services.

#### **Requirements**:
1. **Hardware**:
   - **Small Deployment**: 2 CPUs, 8 GB RAM, 50 GB SSD.
   - **Large Deployment**: 4 CPUs, 16 GB RAM, 100 GB SSD for high trace volumes.

2. **Storage**:
   - Use persistent storage (e.g., Cassandra, Elasticsearch, or S3) for long-term trace retention.

3. **Redundancy**:
   - Deploy in a distributed setup for fault tolerance and scalability.

4. **Networking**:
   - Open ports for trace ingestion (default: `14250` for Jaeger or `4317` for OpenTelemetry).

#### **Scaling**:
- Add more nodes to handle higher trace ingestion rates.

---

### **5. Alertmanager: Notification Engine**
Alertmanager processes alerts from Prometheus and routes them to the appropriate teams.

#### **Requirements**:
1. **Hardware**:
   - **Small Deployment**: 1 CPU, 2 GB RAM.
   - **Large Deployment**: 2 CPUs, 4 GB RAM.

2. **Redundancy**:
   - Deploy two Alertmanager instances in active-active mode for high availability.

3. **Networking**:
   - Open port `9093` for Prometheus to send alerts.

#### **Scaling**:
- Typically, Alertmanager doesn’t require scaling unless handling extremely high alert volumes.

---

### **6. OpenTelemetry Collector: Telemetry Hub**
The OpenTelemetry Collector collects data from your applications and forwards it to Prometheus, Loki, or Jaeger.

#### **Requirements**:
1. **Hardware**:
   - **Small Deployment**: 2 CPUs, 4 GB RAM.
   - **Large Deployment**: 4 CPUs, 8 GB RAM for high telemetry volumes.

2. **Networking**:
   - Open ports for telemetry ingestion (default: `4318` for OTLP HTTP and `4317` for OTLP gRPC).

#### **Scaling**:
- Deploy multiple collectors behind a load balancer for horizontal scaling.

---

### **High-Level Infrastructure Plan**

| **Component**         | **Instance Count** | **Hardware (Per Instance)**        | **Storage**                       |
|-----------------------|--------------------|------------------------------------|-----------------------------------|
| **Prometheus**        | 2                 | 4 CPUs, 16 GB RAM, 100 GB SSD      | Thanos or S3 for long-term metrics|
| **Loki**              | 2                 | 4 CPUs, 16 GB RAM, 200 GB SSD      | S3-compatible storage             |
| **Grafana**           | 1–2             | 4 CPUs, 8 GB RAM, 50 GB SSD         | N/A                               |
| **Jaeger/Tempo**      | 2                 | 4 CPUs, 16 GB RAM, 100 GB SSD      | Cassandra, Elasticsearch, or S3   |
| **Alertmanager**      | 2                 | 2 CPUs, 4 GB RAM                   | N/A                               |
| **OpenTelemetry**     | 2                 | 4 CPUs, 8 GB RAM                   | N/A                               |

---

### **Supporting Infrastructure**

1. **Load Balancers**:
   - Use HAProxy or Nginx for distributing traffic to Prometheus, Grafana, Loki, and OpenTelemetry Collectors.

2. **Storage**:
   - Use object storage (e.g., S3, MinIO) for logs, metrics, and traces.
   - For on-premise setups, consider Ceph or GlusterFS.

3. **Networking**:
   - Ensure all components can communicate securely.
   - Use firewalls and VPNs to secure data flows.

4. **Kubernetes (Optional)**:
   - Deploy the stack in a Kubernetes cluster for easier scaling and management.
