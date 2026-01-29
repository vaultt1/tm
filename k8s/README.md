```bash
kubectl get svc -n task-management
```

```bash
sudo apt update
sudo apt install nginx -y
```

```bash
sudo rm /etc/nginx/sites-available/default
```

```bash
sudo nano /etc/nginx/sites-available/default
```

```bash
server {
    listen 80;

    server_name _;  # catch all

    location / {
        proxy_pass http://192.168.49.2:30995;  # Minikube IP + NodePort
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo nginx -t
```

```bash
sudo systemctl reload nginx
```



