# DevSecOps Tools & Recommendations

---

## 1️⃣ Source Control & Collaboration (SCM)

**Purpose:** Version control & team collaboration

- **Git & Repo Management**
  - Git (already in use)

---

## 2️⃣ CI/CD Pipeline

**Purpose:** Automate build, test, and deployment

- **GitHub Actions** ✅ (ideal if using GitHub)
- **Jenkins** (enterprise/on-prem) – widely used in the industry

---

## 3️⃣ Code Quality & Testing

**Purpose:** Ensure reliable, maintainable, and well-tested code

### Static Code Analysis

- **SonarQube** – Analyzes code quality and provides actionable insights

### Unit, Integration & API Testing

- **Jest (FE + BE)** – Primary testing tool
  - Backend unit tests
  - Backend integration tests
  - Frontend component tests

- **Supertest (API)** – For backend API testing
  - Test routes, auth, validation, error handling
  - Runs inside Jest

- **Smoke Test Script** – Minimal, production-style tests
  - Hits `/health` endpoint
  - Hits one critical API
  - Fails fast if broken

**Benefits:**

- Industry credibility
- Simple setup
- Fast CI
- Minimal maintenance

---

## 4️⃣ Dependency & Supply Chain Security

- **Snyk** – Detects vulnerabilities in dependencies, containers, and code **before production**

---

## 5️⃣ Container Security

- **Trivy** – Scans containers, filesystems, and Kubernetes manifests for vulnerabilities & misconfigurations

**Usage in SaaS teams:**

- Trivy → Container & infra security
- Snyk → Dependency security
- They complement each other

---

## 6️⃣ Kubernetes Security

### Policy & Admission Control

- **OPA Gatekeeper** – Enforces security rules before resource creation
  🔐 “Kubernetes firewall for bad configs”

- **Kyverno** – Enforces security policies via YAML
  🔐 “kubectl apply, but with security guardrails”
  ✅ Recommended for current use

### Runtime Security

- **Falco** – Runtime security engine
  Monitors:
  - System calls (execve, open, write, etc.)
  - Container activity
  - Pod behavior

  Alerts on:
  - `kubectl exec` into pods
  - Privilege escalation
  - Crypto miners or suspicious processes
  - Unexpected network connections

### Kubernetes Scanning Tools

- **kube-bench** – CIS benchmark checks
- **kube-hunter** – Vulnerability scanning

---

## 7️⃣ Secrets Management

- **Kubernetes Secrets + Sealed Secrets**
  - Minimal setup
  - GitOps-friendly
  - Cloud-agnostic

---

## 8️⃣ Infrastructure as Code (IaC)

- **Terraform + Helm** – Simple, templated, GitOps-friendly K8s deployments

### IaC Security Scanning

- **Checkov** – For Terraform + Helm charts
- **tfsec** – Simple, Terraform-only scans

---

## 9️⃣ DAST (Dynamic Application Security Testing)

- **OWASP ZAP** – Runtime web security testing

---

## 🔟 Observability & Monitoring

### Metrics & Logs

- **Prometheus** – Metrics
- **Grafana** – Dashboards
- **ELK Stack** – Logs

### Security Monitoring (SIEM)

- **Wazuh** – Cost-effective, Kubernetes-friendly, ideal for small-medium SaaS
- **Elastic Security** – Best if ELK stack is already in use

---

## 1️⃣1️⃣ Release & Deployment Strategy

- **Argo CD** – GitOps deployment tool

---

## 1️⃣2️⃣ DevSecOps Maturity Model

| Level           | Tools/Skills                                        |
| --------------- | --------------------------------------------------- |
| 🟢 Beginner     | Docker, Git, Basic K8s YAML                         |
| 🟡 Intermediate | CI/CD, Trivy, Snyk, SonarQube                       |
| 🔴 Advanced     | GitOps (ArgoCD), Vault, OPA Gatekeeper, Falco, SIEM |

---
