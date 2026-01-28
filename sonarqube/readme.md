## **SonarQube Installation**

```bash
docker run -itd --name sonarqube-server -p 9000:9000 sonarqube:lts-community
```

Access SonarQube via:  
[SonarQube URL](http://<VM-IP>:9000)

**Monitor regularly:**

```bash
htop
docker stats
```

---

### **Trivy Installation**

**On bash:**

```bash
sudo apt update
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

**Some Trivy Commands:**

```bash
trivy image <image name>
trivy fs --security-checks vuln,config <folder name or path>
```

---

### **Jenkins Plugins Installation**

1. Go to Jenkins URL → `Manage Jenkins` → `Plugins` → `Available Plugins`
2. Install the following plugins:
   - SonarQube Scanner
   - SonarQube Quality Gate
   - OWASP Dependency Check
   - Docker (Install all plugins)
3. Reload Jenkins.

---

### **Jenkins and SonarQube Integration**

1. In SonarQube UI → `Administration` → `Configurations` → `Webhook`
   - Create a new webhook:
     - **Name:** Any name
     - **URL:** `Jenkins URL /sonarqube-webhook/`
   - Click `Create`.

2. In SonarQube UI → `Security` → `Users` → `Create Token`
   - Generate and copy the token.

---

### **Jenkins Configuration**

1. In Jenkins UI → `Manage Jenkins` → `Credentials` → `Global` → `Add Credentials`
   - **Secret text:** Paste the token copied from SonarQube
   - **ID:** `sonar`
   - Click `Create`.

2. In Jenkins UI → `Manage Jenkins` → `System` → `SonarQube Server`
   - Add a new SonarQube server:
     - **Name:** `sonar`
     - **URL of SonarQube:** Enter SonarQube URL
     - **Server Authentication Token:** Select `sonar` from credentials
   - Click `Add`.

---

### **SonarQube Quality Gate Setup**

1. In Jenkins UI → `Manage Jenkins` → `Tools` → `SonarQube Scanner Installations`
   - Add a new SonarQube scanner:
     - **Name:** `sonar`
     - **Select any version**
     - Click `Save`.

---

### **OWASP Dependency Check Setup**

1. In Jenkins UI → `Manage Jenkins` → `Tools` → `Dependency Check`
   - Add a new Dependency Check tool:
     - **Name:** `owasp`
     - **Install Automatically**
     - Add installer → From `github.com` (latest version)
   - Click `Save`.
