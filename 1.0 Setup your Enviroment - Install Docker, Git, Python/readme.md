
# 🧰 Setup Your Environment (Ubuntu VM)

This guide will walk you through setting up a reliable development and testing environment on an Ubuntu VM for our 3-month Cloud Native journey. This VM will host:
- [x]  Our FastAPI web application [See Project Repo](https://github.com/ChisomJude/student-project-tracker) ✅
- [x]  Dockerized services ✅
- [x]  Kubernetes cluster (Kind) ✅
- [x]  GitOps setup (ArgoCD) ✅
- [x]  Monitoring and observability stack (Prometheus, Grafana, Loki) ✅

---

## ☁️ Recommended VM Capacity

For a smooth experience, especially when running Kind clusters and monitoring tools:

| Resource     | Recommended Minimum |
|--------------|---------------------|
| **CPU**      | 2 vCPUs             |
| **RAM**      | 4 GB (8 GB ideal)   |
| **Storage**  | 30 GB SSD           |
| **OS**       | Ubuntu 20.04 or 22.04 LTS |

---

## 📦 Step-by-Step Setup Instructions

### 1. Update and Upgrade System
```bash
sudo apt update && sudo apt upgrade -y
```
### 2. Install Docker
```bash

sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```
***⚠️ Log out and back in for the Docker group to apply.***

### 3. Install Git
```bash
sudo apt install -y git
```

### 4.  Install Python 3 & Pip (for local testing)
```bash
sudo apt install -y python3 python3-pip
```

### 8.  Optional but helpful
- [x] Install Visual Studio Code on your Local PC (for Remote SSH extension)
- [x] Install tmux on your VM, for managing multiple sessions

```
sudo apt install -y tmux
```

#### Other useful commands with tmux
```
# start a session
tmux new -s mysession

#view all sessions
tmux ls

#attach a session
tmux attach -t mysession
