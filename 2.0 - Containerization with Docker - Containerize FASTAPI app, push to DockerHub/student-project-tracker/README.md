# Student-Project-Tracker Web APP
A simple FastAPI web application for registering students and tracking their weekly progress during the Cloud Native Series.

### Key Features:
- Register new students (generates a unique ID).
- Track weekly progress for each student.
- All students use one central MongoDB (hosted on MongoDB Atlas or similar).
- Simple endpoints for registration, status check, and progress update.

## 📦 Prerequisites
- Python 3.10+
- Git
- MongoDB Atlas account (to get your connection string)

---

## 💻 Local Development Setup

### 1. Clone the Repository
```bash
git clone https://github.com/chisomjude/student-project-tracker.git
cd student-project-tracker
```

### 2. Create Virtual Environment & Install Dependencies
```bash
python3 -m venv venv

# Linux
source venv/bin/activate  # On Windows use: venv\Scripts\activate
. venv/bin/activate

pip install -r requirements.txt
```

### 3.Db Conenctions
- create a .env file 
```
VAULT_ADDR=<insert the right value here>
VAULT_ROLE_ID=<insert the right value here>
VAULT_SECRET_ID=<insert the right value here>

# Caution: Ensure there is no space after each value. 
```

### 4. Run the Application Locally
```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
Visit `http://vmip:8000` to see your app in action.

---

## 🐳 Docker Instructions

### 1. Build Docker Image
```bash
docker build -t student-tracker .
```

### 2. Run Docker Container
```bash
docker run --env-file .env -p 8000:8000 student-tracker
```

### 3. Push to Docker Hub
Ensure you're logged in:
```bash
docker login
```
Tag and push your image:
```bash
docker tag student-tracker your-dockerhub-username/student-tracker

docker push your-dockerhub-username/student-tracker
```

---

## 📬 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST   | `/register?name=YourName` | Register new student |
| GET    | `/status/{student_id}`    | View registration and progress |
| POST   | `/update/{student_id}?week=week1` | Update progress by week |

---

### To run the applicationa as a service...
```bash
sudo vi /etc/systemd/system/student-tracker.service 
# The <student-tracker.service>, you can choose any other name you like.
```

```bash
# The service file content...
[Unit]
Description=Student Tracker Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/ubuntu/Devops-CloudNative---HandsOn/2.0 - Containerization with Docker - Containerize FASTAPI app, push to DockerHub/student-project-tracker
ExecStartPre=-/usr/bin/docker stop student-tracker
ExecStartPre=-/usr/bin/docker rm student-tracker
ExecStart=/usr/bin/docker run -d \
    --name student-tracker \
    -p 8000:8000 \
    --env-file .env \
    dolatunj1012/student-tracker:latest
ExecStop=/usr/bin/docker stop student-tracker
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```
```bash
# Enable and start service:
1. sudo systemctl daemon-reload   # Reload systemd
2. sudo systemctl enable student-tracker # Enable service (start on boot)
3. sudo systemctl start student-tracker  # Start service now.
4. sudo systemctl status student-tracker # Check status.
5. sudo journalctl -u student-tracker -f # View logs. 
```

## 🌐 Deploying to Cloud (Optional)
You can deploy the app on platforms like:
- Render
- Railway
- Fly.io
- Azure App Service
- Elastic Beanstalk or more


## 👩🏽‍💻 Built for the Cloud Native Series by Chisom
This project is used for learning cloud-native tools and Handson-Project.

<hr>

#### Deployment on Elastic Beanstalk
You will need to follow the steps listed below to be able to deploy the docker application to Elastic Beanstalk. 

## Step 1: Install EB CLI
```bash
 # Install EB CLI
 pip install awsebcli

 # Very installation
 eb --version
```

## Step 2: Make sure you're in the project directory....
```bash
cd "/home/ubuntu/Devops-CloudNative---HandsOn/2.0 - Containerization with Docker - Containerize FASTAPI app, push to DockerHub/student-project-tracker"
```

Inside the app/main.py, add the information below. Just to use it to confirm if our app is running perfectly.

```bash
# Add this to your app/main.py

@app.get("/health")
async def health_check():
    return {"status": "healthy", "message": "Student Tracker API is running"}

```

### Create Dockerrun.aws.json - This is useful to create configuration for Beanstalk. 

```bash
nano Dockerrun.aws.json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "dolatunj1012/student-tracker:latest",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "8000",
      "HostPort": "80"
    }
  ],
  "Environment": [
    {
      "Name": "VAULT_ADDR",
      "Value": <insert your VAULT_ADDR HERE>
    },
    {
      "Name": "VAULT_ROLE_ID", 
      "Value": <insert your VAULT_ROLE_ID HERE>
    },
    {
      "Name": "VAULT_SECRET_ID",
      "Value": <insert your VAULT_SECRET_ID HERE>
    }
  ]
}

Alternatively, if you dont want to hardcode secrets in the Dockerrun.aws.json file

# Set environment variables via EB CLI
eb setenv VAULT_ADDR=<insert your VAULT_ADDR HERE>
eb setenv VAULT_ROLE_ID=<insert your VAULT_ADDR HERE>
eb setenv VAULT_SECRET_ID=<insert your VAULT_ADDR HERE>

# Deploy changes
eb deploy
```


### Step 4: Configure AWS Credentials
```bash
# Configure AWS CLI (if not already done)
aws configure

# You'll need:
# AWS Access Key ID
# AWS Secret Access Key  
# Default region (e.g., us-east-1)
# Default output format (json)
```

#### Step 5: Initialize Elastic Beanstalk
```bash
# Initialize EB application
eb init

# Follow the prompts:
# 1. Select region (e.g., us-east-1)
# 2. Choose "Create new application" 
# 3. Application name: student-tracker
# 4. Platform: Docker
# 5. Platform version: Docker running on 64bit Amazon Linux 2023
# 6. Do you want to set up SSH: Yes (recommended)


Expected output
Application student-tracker has been created
```

### Step 6: Create Environment
```bash
# Create production environment
eb create student-tracker-prod

# This will:
# - Create EC2 instances
# - Set up load balancer
# - Configure auto-scaling
# - Deploy your application

Expected output:
   Creating application version archive "app-241201_123456".
   Uploading student-tracker/app-241201_123456.zip to S3...
   Creating environment "student-tracker-prod"...
```


#### Add On: Monitoring Part
```bash
# Check environment status
eb status

# View logs
eb logs

# Open application in browser
eb open
```