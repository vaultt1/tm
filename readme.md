# Task Management Application

This repository contains a simple Task Management Application with a Node/Express backend and a React frontend, fully containerized with Docker and Kubernetes support.


## Prerequisites ✅

- Node.js v22.14.0 (or compatible Node 22.x) - for local development
- npm (bundled with Node) - for local development
- Docker & Docker Compose - for containerized deployment
- Minikube - for local Kubernetes cluster
- kubectl - Kubernetes command-line tool

---

## 🐳 Docker Setup

### Step 1: Build Docker Images

Build the Docker images for both backend and frontend:

```bash
# Build backend image
cd Backend
docker build -t someone15me/dp:backend-latest .

# Build frontend image
cd ../Frontend
docker build -t someone15me/dp:frontend-latest --build-arg REACT_APP_BACKEND_URL=http://localhost:8082 .
cd ..
```

### Step 2: Login to Docker Hub

```bash
docker login -u <username> -p <pass>
```

### Step 3: Push Images to Docker Hub

```bash
# Push backend image
docker push someone15me/dp:backend-latest

# Push frontend image
docker push someone15me/dp:frontend-latest
```

### Step 4: Run with Docker Compose (Local Testing)

From the repository root, run:

```bash
docker-compose up --build
```

This will:
- Build and start the backend on port `8082`
- Build and start the frontend on port `3000`
- Create a shared network for container communication

Access the application:
- Frontend: http://localhost:3000
- Backend API: http://localhost:8082

To stop:
```bash
docker-compose down
```

---

## ☸️ Kubernetes Setup with Minikube

### Step 1: Install Minikube and kubectl

**On Linux:**
```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**On macOS:**
```bash
# Using Homebrew
brew install kubectl minikube
```

**On Windows:**
```powershell
# Using Chocolatey
choco install minikube kubernetes-cli
```

### Step 2: Start Minikube

```bash
# Start Minikube cluster
minikube start

# Verify cluster is running
kubectl cluster-info

# Enable Minikube's Docker daemon (so you can use local images)
eval $(minikube docker-env)
```

### Step 3: Build Images in Minikube

Since we're using Minikube, you can either:
- **Option A:** Use images from Docker Hub (recommended)
- **Option B:** Build images directly in Minikube's Docker environment

**Option A - Using Docker Hub (Recommended):**
```bash
# Make sure images are pushed to Docker Hub (see Docker Setup Step 3)
# Then proceed to Step 4
```

**Option B - Build in Minikube:**
```bash
# Enable Minikube Docker daemon
eval $(minikube docker-env)

# Build images
cd Backend
docker build -t someone15me/dp:backend-latest .
cd ../Frontend
docker build -t someone15me/dp:frontend-latest --build-arg REACT_APP_BACKEND_URL=http://localhost:8082 .
cd ..
```

### Step 4: Deploy to Kubernetes

Deploy the application to your Minikube cluster:

```bash
# Create namespace
kubectl apply -f k8s/namespace.yaml

# Deploy the application (creates 2 replicas, each with 2 containers: frontend + backend)
kubectl apply -f k8s/deployment.yaml

# Create service to expose the application
kubectl apply -f k8s/service.yaml
```

### Step 5: Verify Deployment

```bash
# Check namespace
kubectl get namespace task-management

# Check pods (should show 2 pods, each with 2 containers)
kubectl get pods -n task-management

# Check deployment
kubectl get deployment -n task-management

# Check service
kubectl get service -n task-management

# View detailed pod information
kubectl describe pods -n task-management
```

### Step 6: Access the Application

**Get Minikube IP:**
```bash
minikube ip
```

**Access the application:**
- Frontend: `http://<minikube-ip>:30080`
- Backend API: `http://<minikube-ip>:30082`

**Or use Minikube service command:**
```bash
# Open frontend in browser
minikube service task-management-service -n task-management --url

# Or use port-forward for direct access
kubectl port-forward -n task-management service/task-management-service 3000:3000 8082:8082
```

Then access:
- Frontend: http://localhost:3000
- Backend: http://localhost:8082

---

## 📋 Kubernetes Architecture

### Deployment Structure

- **Replicas:** 2 pods for high availability
- **Containers per Pod:** 2 containers (frontend + backend) sharing the same network namespace
- **Service Type:** NodePort (exposes on ports 30080 for frontend, 30082 for backend)

### Pod Communication

Since both containers are in the same pod, they share:
- Network namespace (can communicate via `localhost`)
- Storage volumes (backend data is stored in a shared volume)

### High Availability

- 2 replica pods ensure that if one pod fails, the other continues serving traffic
- The service automatically load-balances between the pods

---

## 🔧 Useful Kubernetes Commands

### View Logs

```bash
# View logs from all containers in a pod
kubectl logs -n task-management <pod-name>

# View logs from specific container
kubectl logs -n task-management <pod-name> -c backend
kubectl logs -n task-management <pod-name> -c frontend

# Follow logs
kubectl logs -n task-management <pod-name> -f
```

### Scale Deployment

```bash
# Scale to 3 replicas
kubectl scale deployment task-management-app -n task-management --replicas=3

# Check scaled pods
kubectl get pods -n task-management
```

### Update Deployment

```bash
# After pushing new images to Docker Hub, update the deployment
kubectl set image deployment/task-management-app \
  backend=someone15me/dp:backend-latest \
  frontend=someone15me/dp:frontend-latest \
  -n task-management

# Or apply the deployment again
kubectl apply -f k8s/deployment.yaml
```

### Delete Resources

```bash
# Delete all resources
kubectl delete -f k8s/

# Or delete individually
kubectl delete deployment task-management-app -n task-management
kubectl delete service task-management-service -n task-management
kubectl delete namespace task-management
```

### Debugging

```bash
# Execute command in a container
kubectl exec -n task-management <pod-name> -c backend -- ls /app

# Get shell access
kubectl exec -n task-management <pod-name> -c backend -it -- /bin/sh

# Describe resources for troubleshooting
kubectl describe pod <pod-name> -n task-management
kubectl describe deployment task-management-app -n task-management
```

---

## 🏃 Run Locally (Development)

### 1) Start the backend

1. Open a terminal and go to the `Backend` folder:

```bash
cd Backend
```

2. Install dependencies and start the server:

```bash
npm install
npm start
```

3. The backend listens on port 8082 by default. Verify the server is up:

```
GET http://localhost:8082/health
```

> Optional `Backend/.env` variables:
> - `SERVER_PORT` (default: `8082`)
> - `FRONTEND_URL` (default: `*`)
> - `SEED_DB` (set to `false` to skip inserting sample tasks)

### 2) Start the frontend

1. Create a `.env` file in the `Frontend` folder and add the backend URL:

```
REACT_APP_BACKEND_URL=http://localhost:8082
```

2. Install dependencies and start the dev server:

```bash
cd Frontend
npm install
npm start
```

3. The React app runs by default on http://localhost:3000

---

## 🧪 Tests & Scripts

- Backend smoke test:
- Run the following command only after starting the backend server.

```bash
cd Backend
npm run smoke-test
```

---

## 📝 Notes

- **Docker Images:** Images are stored on Docker Hub under `someone15me/dp`
- **Kubernetes Namespace:** All resources are deployed in the `task-management` namespace
- **Ports:** 
  - Frontend: 3000 (container) → 30080 (NodePort)
  - Backend: 8082 (container) → 30082 (NodePort)
- **Storage:** Backend data is stored in an `emptyDir` volume (ephemeral, lost on pod deletion)
- **High Availability:** 2 replica pods ensure service continuity
- **Multi-Container Pod:** Frontend and backend share the same pod, enabling efficient localhost communication

---

## 🚀 Quick Start Summary

**Docker:**
```bash
docker-compose up --build
```

**Kubernetes:**
```bash
minikube start
kubectl apply -f k8s/
minikube service task-management-service -n task-management --url
```

---

## 🔐 Docker Hub Credentials

- **Username:** someone15me
- **Repository:** dp
- **Images:** 
  - `someone15me/dp:backend-latest`
  - `someone15me/dp:frontend-latest`
