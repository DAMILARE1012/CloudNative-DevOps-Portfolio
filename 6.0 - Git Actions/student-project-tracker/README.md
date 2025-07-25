# Student-Project-Tracker Web APP 🚀

A modern FastAPI web application for registering students and tracking their weekly progress during the Cloud Native Series. This project demonstrates a complete DevOps pipeline with containerization, CI/CD automation, and cloud deployment.

*Last updated: July 25, 2025 - ECS permissions configured*

## 🌟 **Live Application**
**🔗 Production URL**: [http://student-tracker-alb-1839289530.us-east-1.elb.amazonaws.com](http://student-tracker-alb-1839289530.us-east-1.elb.amazonaws.com)  
**📚 API Documentation**: [http://student-tracker-alb-1839289530.us-east-1.elb.amazonaws.com/docs](http://student-tracker-alb-1839289530.us-east-1.elb.amazonaws.com/docs)

---

## 🎯 **DevOps Project Objectives Completed**

✅ **Repository Setup**: Recreated from [original repo](https://github.com/ChisomJude/student-project-tracker) with enhanced DevOps features  
✅ **Local Development**: Application tested locally with Vault credentials  
✅ **CI/CD Pipeline**: GitHub Actions workflow for automated testing, building, and deployment  
✅ **Cloud Deployment**: Advanced AWS ECS deployment (upgraded from basic EC2)  
✅ **Production Ready**: Live application with load balancing and auto-scaling capabilities  

---

## 🏗️ **Architecture Overview**

### **Deployment Flow:**
```
GitHub Push → GitHub Actions → Build & Test → Push to ECR → Deploy to ECS → Health Checks → Live!
```

---

## 🚀 **Key Features**

### **Application Features:**
- � Register new students (generates unique ID)
- 📈 Track weekly progress for each student  
- 🌐 Centralized MongoDB database
- 🔒 Secure credential management via Vault
- 📊 Simple REST API with automatic documentation

### **DevOps Features:**
- 🔄 **Automated CI/CD**: GitHub Actions pipeline
- 🐳 **Containerized**: Docker with optimized builds
- ☁️ **Cloud Native**: AWS ECS with Fargate
- 🔍 **Health Monitoring**: Automated health checks
- 📈 **Scalable**: Auto-scaling containers
- 🔐 **Secure**: Vault secrets + IAM roles

---

## 💻 **Local Development Setup**

### **Prerequisites:**
- Python 3.10+
- Docker & Docker Compose
- Git
- MongoDB Atlas account
- HashiCorp Vault access

### **1. Clone & Setup Repository**
```bash
git clone https://github.com/DAMILARE1012/CloudNative-DevOps-Portfolio.git
cd "CloudNative-DevOps-Portfolio/6.0 - Git Actions/student-project-tracker"
```

### **2. Environment Configuration**
Create `.env` file with Vault credentials:
```bash
# Vault Configuration
VAULT_ADDR=http://44.204.193.107:8200
VAULT_ROLE_ID=your-role-id
VAULT_SECRET_ID=your-secret-id
```

### **3. Local Development with Python**
```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Export Vault credentials
export VAULT_ADDR=http://44.204.193.107:8200
export VAULT_ROLE_ID=your-role-id
export VAULT_SECRET_ID=your-secret-id

# Run application
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### **4. Local Development with Docker**
```bash
# Build Docker image
docker build -t student-tracker .

# Run with environment file
docker run --env-file .env -p 8000:8000 student-tracker

# Visit: http://localhost:8000
```

---

## 🔄 **CI/CD Pipeline**

### **GitHub Actions Workflow:**
Located in `.github/workflows/student-tracker-ci_cd.yml`

### **Pipeline Stages:**

#### **1. 🧪 Test Stage**
- Python 3.12 environment setup
- Dependency installation
- Unit tests execution
- Code quality checks

#### **2. 🏗️ Build Stage**
- Docker image build
- Multi-architecture support
- Push to Amazon ECR
- Image vulnerability scanning

#### **3. 🚀 Deploy Stage** (Production branch only)
- ECS task definition update
- Rolling deployment to ECS service
- Zero-downtime deployment
- Service stability validation

#### **4. ✅ Health Check Stage**
- Application health verification
- API endpoint smoke tests
- Performance benchmarking
- Deployment success notification

### **Trigger Conditions:**
```yaml
# Automatic triggers
- Push to: development, production branches
- Path filter: 6.0 - Git Actions/student-project-tracker/**
- Pull requests to protected branches
```

---

## ☁️ **AWS ECS Setup Guide**

### **Prerequisites:**
- AWS Account with programmatic access
- AWS CLI configured with appropriate permissions
- Docker installed locally

### **Step 1: Create ECR Repository**
```bash
# Login to AWS
aws configure

# Create ECR repository
aws ecr create-repository --repository-name student_tracker --region us-east-1

# Get login command and execute
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

### **Step 2: Build and Push Docker Image**
```bash
# Build your Docker image
docker build -t student_tracker .

# Tag for ECR
docker tag student_tracker:latest <account-id>.dkr.ecr.us-east-1.amazonaws.com/student_tracker:latest

# Push to ECR
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/student_tracker:latest
```

### **Step 3: Create ECS Cluster**
**Via AWS Console:**
1. **ECS Console** → **Clusters** → **Create Cluster**
2. **Cluster name**: `student_tracker_cluster`
3. **Infrastructure**: AWS Fargate (serverless)
4. **Monitoring**: Enable Container Insights
5. **Create**

### **Step 4: Create Task Definition**
**Via AWS Console:**
1. **ECS Console** → **Task definitions** → **Create new task definition**
2. **Task definition family**: `student_tracker_task`
3. **Launch type**: AWS Fargate
4. **Operating system**: Linux/X86_64
5. **CPU**: 0.25 vCPU, **Memory**: 0.5 GB

**Container Definition:**
- **Container name**: `student_tracker_container`
- **Image URI**: `<account-id>.dkr.ecr.us-east-1.amazonaws.com/student_tracker:latest`
- **Port mappings**: Container port `8000`, Protocol `TCP`

**Environment Variables:**
```bash
VAULT_ADDR=http://44.204.193.107:8200
VAULT_ROLE_ID=your-role-id
VAULT_SECRET_ID=your-secret-id
```

### **Step 5: Create Application Load Balancer**
**Via AWS Console:**
1. **EC2 Console** → **Load Balancers** → **Create Load Balancer**
2. **Application Load Balancer** → **Create**
3. **Name**: `student-tracker-alb`
4. **Scheme**: Internet-facing
5. **VPC**: Default VPC
6. **Subnets**: Select all available subnets

**Security Group:**
- **Inbound rules**: HTTP (80) from 0.0.0.0/0
- **Outbound rules**: All traffic

**Target Group:**
- **Target type**: IP addresses
- **Protocol**: HTTP, **Port**: 8000
- **Health check path**: `/docs`

### **Step 6: Create ECS Service**
**Via AWS Console:**
1. **ECS Console** → **Clusters** → **student_tracker_cluster** → **Services** → **Create**
2. **Environment**: AWS Fargate
3. **Task definition**: `student_tracker_task:latest`
4. **Service name**: `student_tracker_service`
5. **Desired tasks**: 1

**Load balancing:**
- **Load balancer type**: Application Load Balancer
- **Load balancer**: `student-tracker-alb`
- **Container**: `student_tracker_container 8000:8000`
- **Target group**: Create new or select existing

### **Step 7: Configure IAM Permissions**
**Required IAM Policies for GitHub Actions:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:*",
                "ecs:*",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}
```

**Attach to your IAM user:**
- `AmazonECS_FullAccess`
- `AmazonEC2ContainerRegistryPowerUser`

### **Step 8: GitHub Actions Secrets**
**Add these secrets to your GitHub repository:**
- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key

### **Step 9: Verify Deployment**
```bash
# Check ECS service status
aws ecs describe-services --cluster student_tracker_cluster --services student_tracker_service

# Get load balancer DNS
aws elbv2 describe-load-balancers --names student-tracker-alb
```

### **Troubleshooting Common Issues:**

#### **1. Task Fails to Start:**
- Check CloudWatch logs: `/ecs/student_tracker_task`
- Verify environment variables are set correctly
- Ensure Docker image exists in ECR

#### **2. Health Check Failures:**
- Verify target group health check path (`/docs`)
- Check security group allows traffic on port 8000
- Ensure application responds on `/docs` endpoint

#### **3. Load Balancer Connection Issues:**
- Security group must allow HTTP (80) from 0.0.0.0/0
- Target group must point to port 8000
- Application must be listening on 0.0.0.0:8000

#### **4. CI/CD Permission Issues:**
- User needs `AmazonECS_FullAccess` policy
- Task execution role needs `AmazonECSTaskExecutionRolePolicy`
- Service-linked role `AWSServiceRoleForECS` must exist

---

## ☁️ **AWS ECS Deployment**

### **Infrastructure Components:**

#### **🐳 ECS Configuration:**
- **Cluster**: `student_tracker_cluster`
- **Service**: `student_tracker_service` 
- **Task Definition**: `student_tracker_task`
- **Launch Type**: AWS Fargate (serverless)
- **CPU**: 0.25 vCPU
- **Memory**: 0.5 GB

#### **🔗 Networking:**
- **Load Balancer**: Application Load Balancer
- **Target Group**: Health checks on `/docs`
- **Security Groups**: HTTP/HTTPS traffic allowed
- **VPC**: Default VPC with public subnets

#### **📦 Container Registry:**
- **ECR Repository**: `student_tracker`
- **Image URI**: `972936183609.dkr.ecr.us-east-1.amazonaws.com/student_tracker:latest`
- **Encryption**: AES-256
- **Lifecycle**: Automated cleanup policies

---

## 🔧 **Advanced Configuration**

### **Environment Variables:**
```bash
# Required for application startup
VAULT_ADDR=http://44.204.193.107:8200    # Vault server URL
VAULT_ROLE_ID=xxx                        # AppRole role ID
VAULT_SECRET_ID=xxx                      # AppRole secret ID
```

---

## 🔍 **Monitoring & Observability**

### **Health Checks:**
- **Load Balancer**: Health checks every 30 seconds
- **Target Group**: `/docs` endpoint monitoring
- **ECS Service**: Container health monitoring
- **CloudWatch**: Application and infrastructure logs

### **Performance Metrics:**
- **Response Time**: < 2 seconds average
- **Availability**: 99.9% uptime target
- **Scaling**: Auto-scaling based on CPU/memory
- **Error Rate**: < 0.1% error threshold

---

## 🛡️ **Security Implementation**

### **Infrastructure Security:**
- 🔐 **IAM Roles**: Least privilege access
- 🛡️ **Security Groups**: Port-specific access control
- 🔒 **VPC**: Private networking with public load balancer
- 📜 **SSL/TLS**: HTTPS-ready configuration

### **Application Security:**
- 🔑 **Vault Integration**: Secure secrets management
- 🚫 **No Hardcoded Secrets**: Environment-based configuration
- 🔍 **Input Validation**: FastAPI automatic validation
- 📊 **Audit Logging**: Request/response logging

---

## 🚀 **Deployment Instructions**

### **Automated Deployment:**
1. **Push to Production Branch:**
   ```bash
   git checkout production
   git add .
   git commit -m "feat: trigger deployment"
   git push origin production
   ```

---
## 👨‍💻 **Built By**

**DAMILARE1012** - Cloud Native DevOps Portfolio Project  
Original concept by **ChisomJude** - Cloud Native Series  

### **Connect:**
- 🐙 GitHub: [@DAMILARE1012](https://github.com/DAMILARE1012)
- 📧 Project: CloudNative-DevOps-Portfolio
- 🌐 Live Demo: [Student Tracker App](http://student-tracker-alb-1839289530.us-east-1.elb.amazonaws.com)

---
