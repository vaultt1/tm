# Task Management Application

Task Management Application with Node/Express backend and React frontend, containerized with Docker and Kubernetes.

## What Each File Does

### Dockerfiles

**Backend/Dockerfile** - Builds the backend container:
- Installs production dependencies
- Copies backend code
- Runs on port 8082
- Creates data directory for SQLite database

**Frontend/Dockerfile** - Builds the frontend container:
- Builds React app for production
- Serves static files using `serve`
- Runs on port 3000
- Connects to backend at localhost:8082

### docker-compose.yaml

Defines how to run both containers together:
- Backend service on port 8082
- Frontend service on port 3000
- Shared network for communication
- Volume for backend data storage

### Jenkinsfile

Automates building and deploying:
- Checks out code from repository
- Builds Docker images with version tags
- Pushes images to Docker Hub
- Deploys to Kubernetes and updates running containers

### Kubernetes Files (k8s/)

**namespace.yaml** - Creates a separate space called "task-management" for the app

**deployment.yaml** - Defines how to run the app:
- Creates 2 copies (replicas) for availability
- Each copy has 2 containers: frontend and backend
- Sets memory and CPU limits
- Configures environment variables

**service.yaml** - Makes the app accessible:
- Exposes frontend on port 30080
- Exposes backend on port 30082
- Load balances traffic between the 2 copies

## Docker Setup

### Build Images

```bash
# Build backend
cd Backend
docker build -t someone15me/dp:backend-latest .

# Build frontend
cd ../Frontend
docker build -t someone15me/dp:frontend-latest --build-arg REACT_APP_BACKEND_URL=http://localhost:8082 .
cd ..
```

### Push to Docker Hub

```bash
docker login -u someone15me -p Ditiss123
docker push someone15me/dp:backend-latest
docker push someone15me/dp:frontend-latest
```

### Run with Docker Compose

```bash
docker-compose up --build
```

Access:
- Frontend: http://localhost:3000
- Backend: http://localhost:8082

Stop:
```bash
docker-compose down
```

## Kubernetes Setup

### Start Minikube

```bash
minikube start
eval $(minikube docker-env)
```

### Deploy Application

```bash
kubectl apply -f k8s/
```

This creates:
- Namespace: task-management
- Deployment: 2 pods with frontend + backend containers
- Service: exposes app on ports 30080 and 30082

### Check Status

```bash
kubectl get pods -n task-management
kubectl get service -n task-management
kubectl get deployment -n task-management
```

### Access Application

Get Minikube IP:
```bash
minikube ip
```

Access:
- Frontend: `http://<minikube-ip>:30080`
- Backend: `http://<minikube-ip>:30082`

Or use port-forward:
```bash
kubectl port-forward -n task-management service/task-management-service 3000:3000 8082:8082
```

Then access:
- Frontend: http://localhost:3000
- Backend: http://localhost:8082

## Useful Commands

### View Logs

```bash
kubectl logs -n task-management <pod-name>
kubectl logs -n task-management <pod-name> -c backend
kubectl logs -n task-management <pod-name> -c frontend
```

### Update Deployment

```bash
kubectl set image deployment/task-management-app \
  backend=someone15me/dp:backend-latest \
  frontend=someone15me/dp:frontend-latest \
  -n task-management
```

### Delete Everything

```bash
kubectl delete -f k8s/
```

## Local Development

### Backend

```bash
cd Backend
npm install
npm start
```

Backend runs on http://localhost:8082

### Frontend

Create `Frontend/.env`:
```
REACT_APP_BACKEND_URL=http://localhost:8082
```

```bash
cd Frontend
npm install
npm start
```

Frontend runs on http://localhost:3000

## Docker Hub

- Username: someone15me
- Repository: dp
- Images: `someone15me/dp:backend-latest`, `someone15me/dp:frontend-latest`
