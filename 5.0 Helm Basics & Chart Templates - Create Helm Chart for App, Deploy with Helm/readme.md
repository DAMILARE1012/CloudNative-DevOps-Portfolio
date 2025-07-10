# 📦 Deploy with Helm

As we progress with understanding deployment in a cloud native environment, deployment with Helm is a **key component**. Helm is the package manager for Kubernetes, like:
- `apt` for Ubuntu
- `yum` for CentOS  
- `pip` for Python

However, Helm does **much more** - it helps you define, install, and manage Kubernetes applications using ***reusable*** packages called charts.

## 🎯 **Learning Goals**
By the end of this week, you will:
- ✅ Understand Helm concepts and architecture
- ✅ Create a production-ready Helm chart
- ✅ Deploy your Student Tracker app using Helm
- ✅ Manage application lifecycle with Helm
- ✅ Troubleshoot common deployment issues

## 📚 **Key Concepts**

| Concept        | Meaning                                                                                    | Example |
| -------------- | ------------------------------------------------------------------------------------------ | ------- |
| **Chart**      | A Helm package that contains all the Kubernetes manifest files (Deployment, Service, etc.) | `student-tracker/` folder |
| **Release**    | An instance of a chart running in a Kubernetes cluster                                     | `student-tracker` in `my-app` namespace |
| **Repository** | A collection of charts available for use (like DockerHub for Helm)                         | `ingress-nginx/ingress-nginx` |
| **Values**     | Configuration parameters used to customize a chart during installation                     | `my-values.yaml` file |
| **Templates**  | Kubernetes manifests with placeholders that get filled by values                          | `deployment.yaml` with `{{ .Values.image.repository }}` |


## 🚀 **Deploy Student Project Tracker App using Helm**

### **Prerequisites**
Before starting, ensure you have:
- ✅ Kubernetes cluster running (Kind/Minikube/Cloud)
- ✅ kubectl configured and working
- ✅ Docker image built and pushed to DockerHub
- ✅ NGINX Ingress Controller installed (from Week 4)

---

### **Step 1: Install Helm on your VM**
```bash
# Download Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# Verify installation
helm version
```

### **Step 2: Create Helm Chart Structure**
```bash
# Create initial chart
helm create student-tracker 
cd student-tracker

# Clean up default templates (we'll create our own)
rm -r ./templates/*

# Create our templates
cd templates
touch deployment.yaml service.yaml serviceaccount.yaml secret.yaml ingress.yaml _helpers.tpl
```

**Important**: Don't skip the `serviceaccount.yaml` - your deployment will fail without it!

### **Step 3: Verify NGINX Ingress Controller** 
```bash
# Check if NGINX Ingress Controller is installed
kubectl get pods -n ingress-nginx
kubectl get ingressclass

# If not installed, install it:
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Only run this if you don't have NGINX Ingress Controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**Note**: If you already have NGINX Ingress Controller running (from Week 4), skip the installation step!

### **Step 4: Configure Your Chart**

#### **4.1 Update Chart.yaml**
```yaml
apiVersion: v2
name: student-tracker
description: A Helm chart for Student Tracker FastAPI application
type: application
version: 0.1.0
appVersion: "latest"
maintainers:
  - name: your-dockerhub-username
    email: your-email@example.com
```

#### **4.2 Create my-values.yaml**
```yaml
# my-values.yaml - Environment-specific overrides
replicaCount: 3

image:
  repository: your-dockerhub-username/student-tracker
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 8000

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: studenttracker.local
      paths:
        - path: /
          pathType: Prefix

# CRITICAL: Health check configuration
livenessProbe:
  httpGet:
    path: /              # Use root endpoint, not /health
    port: http
  initialDelaySeconds: 30  # Give app time to start
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5

# Environment variables for your app
env:
  - name: VAULT_ADDR
    value: "http://44.204.193.107:8200"
  - name: VAULT_ROLE_ID
    value: "your-role-id"
  - name: VAULT_SECRET_ID
    value: "your-secret-id"
```

#### **4.3 Update values.yaml**
Make sure your `values.yaml` has the correct image repository and health check settings:
```yaml
image:
  repository: your-dockerhub-username/student-tracker
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  port: 8000  # Your app runs on port 8000

livenessProbe:
  httpGet:
    path: /    # Use root endpoint
    port: http
  initialDelaySeconds: 30
readinessProbe:
  httpGet:
    path: /
    port: http
  initialDelaySeconds: 10
```

### **Step 5: Deploy Your Helm Chart**

#### **5.1 Deploy the Application**
```bash
# Create namespace and install
helm install student-tracker ./student-tracker \
  -n my-app \
  --create-namespace \
  -f my-values.yaml

# Check deployment status
helm status student-tracker -n my-app
```

#### **5.2 Verify Deployment**
```bash
# Check pods (should show 3 replicas running)
kubectl get pods -n my-app

# Check services
kubectl get services -n my-app

# Check ingress
kubectl get ingress -n my-app

# View application logs
kubectl logs -f deployment/student-tracker -n my-app
```

#### **5.3 Access Your Application**
```bash
# Method 1: Port forwarding (for testing)
kubectl port-forward --address 0.0.0.0 \
  -n my-app service/student-tracker 8080:8000

# Access at: http://YOUR-SERVER-IP:8080

# Method 2: Via Ingress (production)
# Add to /etc/hosts: YOUR-SERVER-IP studenttracker.local
# Access at: http://studenttracker.local
```

---

## 📁 **Final Chart Structure**

```
student-tracker/
├── Chart.yaml              # Chart metadata ✅
├── values.yaml             # Default values ✅
├── my-values.yaml          # Environment overrides ✅
├── charts/                 # Dependencies (empty for now)
└── templates/              # Kubernetes manifests
    ├── deployment.yaml     # App deployment ✅
    ├── service.yaml        # Service configuration ✅
    ├── serviceaccount.yaml # Service account ✅ (REQUIRED!)
    ├── secret.yaml         # Secrets (optional) ✅
    ├── ingress.yaml        # Ingress rules ✅
    └── _helpers.tpl        # Template helpers ✅
```

---

## 🔧 **Essential Helm Commands**

### **Lifecycle Management**
```bash
# Install release
helm install student-tracker ./student-tracker -n my-app -f my-values.yaml

# Upgrade release
helm upgrade student-tracker ./student-tracker -n my-app -f my-values.yaml

# Rollback release
helm rollback student-tracker 1 -n my-app

# Uninstall release
helm uninstall student-tracker -n my-app
```

### **Monitoring & Debugging**
```bash
# List releases
helm list -n my-app

# Get release status
helm status student-tracker -n my-app

# Get release history
helm history student-tracker -n my-app

# View generated manifests (without installing)
helm template student-tracker ./student-tracker -f my-values.yaml

# Debug and show values
helm get values student-tracker -n my-app
```

---

## 🚨 **Common Issues & Solutions**

### **Issue 1: Pods Not Ready**
**Symptoms**: Pods show `0/1 Ready`, restart frequently
**Solution**: 
- Check health check endpoints (use `/` instead of `/health`)
- Add `initialDelaySeconds` to health checks
- Verify environment variables

### **Issue 2: "ServiceAccount not found"**
**Symptoms**: `Error: failed to create: serviceaccounts "student-tracker" not found`
**Solution**: Create `serviceaccount.yaml` template

### **Issue 3: "Connection Refused" when accessing app**
**Symptoms**: Can't access app via port forwarding
**Solution**: 
- Ensure port forwarding binds to `0.0.0.0`: `--address 0.0.0.0`
- Check AWS Security Groups for port access
- Verify service is running: `kubectl get svc -n my-app`

### **Issue 4: "Release already exists"**
**Symptoms**: `cannot re-use a name that is still in use`
**Solution**: 
```bash
# Delete existing release
helm uninstall student-tracker -n my-app
# Then reinstall
helm install student-tracker ./student-tracker -n my-app -f my-values.yaml
```

---

## 🎯 **Testing Your Deployment**

### **Health Checks**
```bash
# Test health endpoint
curl http://localhost:8080/health
curl http://localhost:8080/

# Test API endpoints
curl http://localhost:8080/docs
curl -X POST http://localhost:8080/register -d "name=TestUser"
```

### **Scaling Test**
```bash
# Scale up
helm upgrade student-tracker ./student-tracker \
  -n my-app -f my-values.yaml \
  --set replicaCount=5

# Verify scaling
kubectl get pods -n my-app
```

---

## 🏆 **Success Criteria**

You've successfully completed Week 5 when you can:
- ✅ Create a complete Helm chart for your application
- ✅ Deploy the application using Helm with custom values
- ✅ Access the application via port forwarding
- ✅ Scale the application using Helm
- ✅ Troubleshoot common deployment issues
- ✅ Use Helm commands for lifecycle management

---

## 📈 **Next Steps**

Congratulations! 🎉 Your application is now deployed using Helm. In Week 6, you'll learn:
- Setting up CI/CD pipelines with GitHub Actions
- Automating Docker builds and Helm deployments
- Implementing GitOps workflows

**Keep your Helm deployment running - we'll use it in the next week!**

