# Kubernetes Files Guide

This folder contains the files needed to run the application in Kubernetes.

## What's in this folder?

- **namespace.yaml** - Creates a separate space for our application
- **deployment.yaml** - Tells Kubernetes how to run our app (2 copies with frontend + backend in each)
- **service.yaml** - Makes the app accessible from outside the cluster

## How to use these files

### Step 1: Create the namespace
This creates a separate area for our application:
```bash
kubectl apply -f namespace.yaml
```

### Step 2: Deploy the application
This starts 2 copies of the app, each with frontend and backend:
```bash
kubectl apply -f deployment.yaml
```

### Step 3: Make it accessible
This creates a service so you can access the app:
```bash
kubectl apply -f service.yaml
```

### Or do all three at once:
```bash
kubectl apply -f .
```

## Check if everything is running

```bash
# See all pods (should show 2 pods)
kubectl get pods -n task-management

# See the service
kubectl get service -n task-management

# See the deployment
kubectl get deployment -n task-management
```

## Access the application

After deploying, you can access:
- **Frontend**: `http://<minikube-ip>:30080`
- **Backend**: `http://<minikube-ip>:30082`

To get your Minikube IP:
```bash
minikube ip
```

Or use port-forward for easier access:
```bash
kubectl port-forward -n task-management service/task-management-service 3000:3000 8082:8082
```
Then access:
- Frontend: http://localhost:3000
- Backend: http://localhost:8082

## View logs

```bash
# See logs from a pod (replace <pod-name> with actual pod name)
kubectl logs -n task-management <pod-name>

# See backend logs
kubectl logs -n task-management <pod-name> -c backend

# See frontend logs
kubectl logs -n task-management <pod-name> -c frontend
```

## Update the application

After pushing new Docker images, update the deployment:
```bash
kubectl set image deployment/task-management-app \
  backend=someone15me/dp:backend-latest \
  frontend=someone15me/dp:frontend-latest \
  -n task-management
```

## Remove everything

To delete all resources:
```bash
kubectl delete -f .
```

Or delete the namespace (this removes everything in it):
```bash
kubectl delete namespace task-management
```

## What each file does

### namespace.yaml
Creates a separate area called "task-management" to keep our app organized.

### deployment.yaml
- Creates 2 copies (replicas) of the application for reliability
- Each copy has 2 containers: frontend and backend
- Sets up resources (memory, CPU limits)
- Configures environment variables

### service.yaml
- Exposes the application so it can be accessed
- Maps ports:
  - Frontend: 3000 (inside) → 30080 (outside)
  - Backend: 8082 (inside) → 30082 (outside)
- Load balances traffic between the 2 copies

## Troubleshooting

**Pods not starting?**
```bash
kubectl describe pod <pod-name> -n task-management
```

**Can't access the app?**
- Check if pods are running: `kubectl get pods -n task-management`
- Check service: `kubectl get service -n task-management`
- Check logs: `kubectl logs <pod-name> -n task-management`

**Need to restart?**
```bash
kubectl rollout restart deployment/task-management-app -n task-management
```
