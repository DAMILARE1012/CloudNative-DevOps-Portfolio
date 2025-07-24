# 📦Case Study 1- Build Your APP (Student tracker App)

This FastAPI app allows students to register, track their weekly progress in the bootcamp, and view others' progress (using a shared MongoDB Atlas backend).

## 🚀 Goal (This Week)
> Dockerize this app, run it locally in a container, and push your image to DockerHub. You should be able to:
- Run the app inside Docker
- Test the API on your browser
- Share the image on DockerHub

---

## ✅ Prerequisites

Ensure the following are already installed in your VM (from [1.0 - Set Up Your Environment](https://github.com/ChisomJude/Hands-on-Devops-CloudNative/tree/master/1.0%20Setup%20your%20Enviroment)):

- Docker  
- Git  
- Python 3 & Pip  
- Kubernetes tools (kubectl, kind)  
  

---

## 🧪 Step-by-Step Guide

### 1. Clone the Project

```bash
sudo su  #become root user
git clone https://github.com/ChisomJude/student-project-tracker.git
cd student-project-tracker
```

### 2. Ensure you can connect to the vault Server

## New - Version
```
New Vault Cred::
We can use the same value this time:

export VAULT_ADDR=http://44.204.193.107:8200
export VAULT_ROLE_ID=f7af58b1-5c22-7c2d-c659-0425d9ce94b2
export VAULT_SECRET_ID=d5f736da-785b-8f5c-9258-48d5d7c43c06


curl http://44.204.193.107:8200/v1/sys/health
```


***We are using a central db, vault credentials  will be provided in the Class Group, if you aren't part of the class you can create yours***


### 3. Run Locally 
To test the app locally before Dockerizing:

```bash
sudo apt install python3-venv -y

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Suppose it fail to start, kindly do the next step...
#Go to: http://<your-VM-public-IP>:8000/docs, ensure Port 8000 is open in the **network security group** - set a new inbound rule, and confirm this works
```
Start the APP locally

![image](https://github.com/user-attachments/assets/aed139e7-9fb2-42a5-a456-1bdd7fd59fc0)


View on the Browser using you Server IP

![image](https://github.com/user-attachments/assets/2cf8a0e7-9cc7-4d0c-8cc2-db6013c55324)

### Output after Registering.
![alt text](image.png)


### 4. Build your image
A Dockerfile is already provided in the project repo; however, feel free to modify it to suit your needs.

✅ Build the image 
<br>✅ and run it locally<br>
✅ Push to your DockerHub 

***Not familiar with docker? Check out this free course at kodekloud - https://learn.kodekloud.com/user/courses/docker-training-course-for-the-absolute-beginner***

```bash
docker login  #enter ur docker cred.
docker build -t yourdockerhubusername/student-tracker:latest .

# on your current terminal where u exported vault env  or you export them again
docker run -d -p 8000:8000 --env-file .env  <dockerubusername>/student-tracker:latest

# Check the app on your browser or curl http://<your-vm-ip>:8000 or confirm your container is running without error docker logs <containerip>
```
![image](https://github.com/user-attachments/assets/2b6cbaec-704f-4ed8-a0bc-345e08764564)


#### Push to DockerHub
Docker file is already on the app folder, feel free to modify to suit ur needs. Signup with http://hub.docker.com/ if you don't have an account


```bash

docker push yourdockerhubusername/student-tracker: latest
```






