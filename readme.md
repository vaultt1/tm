## Local Development

### Backend

```bash
cd Backend
npm install
npm start
```

Backend runs on [http://localhost:8082](http://localhost:8082)

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

Frontend runs on [http://localhost:3000](http://localhost:3000)

---

## 🐳 Step 1: Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

## Add current user to Docker group

```bash
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

Reboot recommended.

---

## 🕸️ Step 2: Install Minikube & kubectl

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl
```

---

## 🎯 Step 3: Install and Set up Jenkins

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```

## Add Jenkins user to Docker group

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### Start Jenkins

```bash
sudo apt install -y kubectl
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

## Increase Jenkins stability (recommended)

Edit:

```bash 
sudo nano /etc/default/jenkins 
```


Add:

```bash 
JAVA_ARGS="-Djenkins.install.runSetupWizard=false -Xms512m -Xmx2048m" 
```

Restart:

```bash 
sudo systemctl restart jenkins 
```

## Fix kubectl context
```bash 
kubectl config use-context minikube
kubectl get nodes
kubectl create namespace task-management

```
---

## ⭐ CRITICAL STEP

```bash
kubectl config view --raw --flatten > kubeconfig.yaml
```

### What this does

* Embeds certs as base64
* Removes references to `/home/<user>/.minikube`
* Makes kubeconfig portable and CI-safe

✅ This is the file Jenkins will use.

---

## 🧩 Configure Jenkins Credentials

In Jenkins UI:

```
Manage Jenkins
→ Credentials
→ (Global)
→ Add Credentials
```

* **Kind:** Secret file
* **ID:** `kubeconfig`
* **File:** `kubeconfig.yaml`
* **Description:** Kubernetes config (inline certs)

Save.

---

## 🔑 Docker Hub Credentials

Add Docker Hub credentials in Jenkins:

* **Kind:** Username with password
* **ID:** `dockerhub-creds`
* **Username:** your Docker Hub username
* **Password:** Docker Hub access token (recommended)

---

### Check Status

```bash
kubectl get pods -n task-management
kubectl get service -n task-management
kubectl get deployment -n task-management
kubectl get all -n task-management
```


pipeline mein security scan stages (OWASP & Trivy) ko catchError() ke andar rakha gaya hai.
Is wajah se agar vulnerabilities milti hain, Jenkins build ko FAIL nahi karta, balki UNSTABLE mark karke pipeline continue kar deta hai.
Deploy stage par koi condition nahi hai jo UNSTABLE build ko rok sake, isliye deployment ho jata hai.

UNSTABLE build ka matlab hota hai application kaam kar rahi hai,
lekin security risks present hain.
Isliye development environment mein allow kar sakte hain,
par production mein deployment strictly block hona chahiye.

hum deployment stage par condition laga denge ki sirf SUCCESS build hi deploy ho.
UNSTABLE ya FAILED build Kubernetes par deploy nahi hoga.


✅ Correct Fix (code bol ke dikha sakta hai):
🔒 Deploy stage gate
```
stage('Deploy to Kubernetes') {
    when {
        expression { currentBuild.result == 'SUCCESS' }
    }
    steps {
        // deployment steps
    }
}
```


UNSTABLE deployment ho raha hai kyunki catchError() pipeline ko fail nahi karta
aur deploy stage par koi restriction nahi hai.
Deployment gate laga ke issue solve ho jata hai.
