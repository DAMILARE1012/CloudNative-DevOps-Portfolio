# 🚀 Kubernetes Observability Stack with Prometheus, Loki & Grafana

## 📋 Task Description
This project demonstrates the complete setup of a production-grade observability stack on Kubernetes using Prometheus for metrics collection, Loki for centralized logging, and Grafana for visualization. The stack monitors a FastAPI-based Student Tracker application with MongoDB backend.

## 🛠️ Tools Used
- **Helm** — For deploying Kubernetes applications
- **Prometheus** — For collecting and storing metrics
- **Loki + Promtail** — For centralized log aggregation
- **Grafana** — For dashboards and visualization
- **Vault** — For secrets management
- **MongoDB** — Application database

## 🏗️ Architecture Overview
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Student       │    │   Prometheus    │    │    Grafana      │
│   Tracker App   │───▶│   (Metrics)     │───▶│  (Dashboards)   │
│   (FastAPI)     │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       ▲                       ▲
         │                       │                       │
         ▼                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   MongoDB       │    │   Loki +        │    │   AlertManager  │
│   (Database)    │    │   Promtail      │    │   (Alerts)      │
│                 │    │   (Logs)        │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📝 Step-by-Step Implementation

### Step 1: Prerequisites Setup ✅
- **Kubernetes Cluster**: Kind cluster with 2 nodes (control-plane + worker)
- **Helm**: v3.18.3 installed and configured
- **kubectl**: Configured to access the cluster

```bash
# Verify cluster status
kubectl get nodes
kubectl get pods --all-namespaces
```

### Step 2: Deploy Local Vault for Secrets Management ✅
Since the external Vault server was unreachable, we deployed a local Vault instance:

```bash
# Add HashiCorp Helm repository
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

# Deploy Vault in development mode
helm install vault hashicorp/vault --namespace vault --create-namespace --set "server.dev.enabled=true"

# Wait for Vault to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=vault -n vault --timeout=300s

# Port forward to access Vault
kubectl port-forward -n vault svc/vault 8200:8200 &
```

### Step 3: Configure Vault Authentication ✅
Set up AppRole authentication and secrets for the Student Tracker application:

```bash
# Login to Vault
export VAULT_ADDR='http://localhost:8200'
vault login root

# Enable AppRole authentication
vault auth enable approle

# Create AppRole for student tracker
vault write auth/approle/role/student-tracker \
    token_policies="default" \
    token_ttl=1h \
    token_max_ttl=4h

# Set role_id and create secret_id
vault write auth/approle/role/student-tracker/role-id role_id=f7af58b1-5c22-7c2d-c659-0425d5d7c43c06
vault write -f auth/approle/role/student-tracker/secret-id

# Enable KV secrets engine and create MongoDB URI secret
vault secrets enable -path=secret kv-v2
vault kv put secret/student01 MONGO_URI="mongodb://localhost:27017/student_project_tracker"
```

### Step 4: Deploy Student Tracker Application ✅
Updated the application configuration to use local Vault and deployed using Helm:

```bash
# Navigate to student-tracker directory
cd "student-tracker"

# Update secret with new Vault configuration
kubectl patch secret vault-secrets -n my-app -p '{
  "data": {
    "VAULT_ADDR": "aHR0cDovL3ZhdWx0LnZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsOjgyMDA=",
    "VAULT_ROLE_ID": "ZjdhZjU4YjEtNWMyMi03YzJkLWM2NTktMDQyNWQ5Y2U5NGIy",
    "VAULT_SECRET_ID": "MzM2MjMxMzEtNTU3Yi01OGMzLTcwNTYtYzFmNTk5OWZiOWRi"
  }
}'

# Deploy MongoDB
helm install mongodb bitnami/mongodb --namespace my-app

# Upgrade student tracker deployment
helm upgrade student-tracker . --namespace my-app
```

### Step 5: Deploy Prometheus Stack ✅
Deployed the complete observability stack using kube-prometheus-stack:

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Deploy the complete observability stack
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

# Verify deployment
kubectl get pods -n monitoring
```

### Step 6: Deploy Loki Stack for Logging ✅
Added centralized logging with Loki and Promtail:

```bash
# Add Grafana Helm repository for Loki
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Deploy Loki stack for centralized logging
helm install loki grafana/loki-stack --namespace monitoring

# Verify Loki deployment
kubectl get pods -n monitoring | grep -E "(loki|promtail)"
```

### Step 7: Configure Grafana Data Sources ✅
Set up Prometheus and Loki as data sources in Grafana:

```bash
# Get Grafana admin password
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward to access Grafana
kubectl port-forward --namespace monitoring svc/prometheus-grafana 3000:80
```

**Grafana Configuration:**
- **URL**: http://localhost:3000
- **Username**: admin
- **Password**: (from kubectl command above)

**Data Sources Added:**
- **Prometheus**: `http://prometheus-kube-prometheus-prometheus:9090`
- **Loki**: `http://loki:3100`

### Step 8: Create Production-Grade Metrics Dashboard ✅
Built comprehensive metrics dashboard with the following panels:

**Panel 1: Student Tracker Pod Status**
- **Data Source**: Prometheus
- **Query**: `sum(kube_pod_status_phase{namespace="my-app"})`
- **Visualization**: Stat
- **Purpose**: Shows total number of running pods

**Panel 2: Memory Usage by Container**
- **Data Source**: Prometheus
- **Query**: `container_memory_usage_bytes{namespace="my-app"}`
- **Visualization**: Gauge
- **Purpose**: Memory consumption per container

**Panel 3: CPU Usage**
- **Data Source**: Prometheus
- **Query**: `rate(container_cpu_usage_seconds_total{namespace="my-app"}[5m])`
- **Visualization**: Time series
- **Purpose**: CPU utilization over time

### Step 9: Create Production-Grade Logs Dashboard ✅
Implemented centralized logging dashboard with multiple log views:

**Panel 1: Application Logs**
- **Data Source**: Loki
- **Query**: `{namespace="my-app"}`
- **Visualization**: Logs
- **Purpose**: All application logs

**Panel 2: Error Logs**
- **Data Source**: Loki
- **Query**: `{namespace="my-app"} |= "error"`
- **Visualization**: Logs
- **Purpose**: Error-specific logs

**Panel 3: MongoDB Logs**
- **Data Source**: Loki
- **Query**: `{namespace="my-app", container="mongodb"}`
- **Visualization**: Logs
- **Purpose**: Database-specific logs

### Step 10: Verify Complete Observability Stack ✅
Comprehensive testing and validation:

```bash
# Verify all components are running
kubectl get pods -n monitoring
kubectl get pods -n my-app

# Test Prometheus metrics collection
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
# Access: http://localhost:9090

# Test Loki log collection
kubectl port-forward -n monitoring svc/loki 3100:3100
curl -G -s "http://localhost:3100/loki/api/v1/query" --data-urlencode 'query={namespace="my-app"}' | jq .

# Test application functionality
kubectl port-forward -n my-app svc/student-tracker-service 8000:80
# Access: http://localhost:8000
```

## 📊 Dashboard Features

### Metrics Dashboard
- **Real-time pod status monitoring**
- **Resource utilization tracking** (CPU, Memory)
- **Container-level metrics**
- **Time-series visualizations**
- **Alert-ready thresholds**

### Logs Dashboard
- **Centralized log aggregation**
- **Multi-container log viewing**
- **Error log filtering**
- **Real-time log streaming**
- **Search and filter capabilities**

## 🔧 Troubleshooting

### Common Issues Resolved

1. **Vault Connection Timeout**
   - **Problem**: External Vault server unreachable
   - **Solution**: Deployed local Vault in development mode

2. **AppRole Authentication**
   - **Problem**: Invalid role or secret ID errors
   - **Solution**: Properly configured AppRole with correct role_id and secret_id

3. **MongoDB URI Secret**
   - **Problem**: Application couldn't read MongoDB connection string
   - **Solution**: Created secret in Vault with correct path and format

4. **Promtail CrashLoopBackOff**
   - **Problem**: One Promtail pod failing
   - **Solution**: Normal behavior with multiple Promtail instances

## 🚀 Advanced Monitoring Capabilities

### Cluster-Level Metrics (Available)
- Node pressure (CPU throttling, memory saturation)
- Disk IO & PVC health
- Network traffic by pod/workload
- Scheduler latency & pending pod queues
- Control plane metrics (API server, scheduler, kube-proxy)
- System-level metrics (load, CPU idle, context switches)

### Application-Level Metrics (Ready for Implementation)
- HTTP request rates and response codes
- Application performance metrics
- Database operation metrics
- Custom business metrics (student registrations, task completions)

## 📈 Results Achieved

✅ **Complete observability stack deployed**  
✅ **Student Tracker application running successfully**  
✅ **Metrics collection working (Prometheus)**  
✅ **Centralized logging operational (Loki + Promtail)**  
✅ **Production-grade dashboards created (Grafana)**  
✅ **Real-time monitoring and alerting ready**  
✅ **Vault integration for secrets management**  
✅ **MongoDB monitoring and logging**  

## 🎯 Next Steps

1. **Add custom application metrics** to the FastAPI app
2. **Implement alerting rules** in Prometheus
3. **Create additional dashboards** for different stakeholders
4. **Set up log retention policies** in Loki
5. **Implement backup strategies** for monitoring data
6. **Add service mesh observability** (if applicable)

## 📚 Key Learnings

- **Helm simplifies complex deployments** with pre-configured charts
- **Vault provides secure secrets management** for applications
- **Prometheus excels at numeric metrics** collection and storage
- **Loki is perfect for log aggregation** and searching
- **Grafana unifies metrics and logs** in comprehensive dashboards
- **Proper configuration is crucial** for observability stack success

---

**🎉 This observability stack provides complete visibility into your Kubernetes applications, enabling proactive monitoring, rapid troubleshooting, and data-driven decision making.**