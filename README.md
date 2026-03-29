#<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/057ebd6c-b876-4bda-b6aa-65aad879d472" />
 Bloodbank-CI-CD-DevOps-project-
Bloodbank-CI/CD-DevOps-project using jenkins sonar trivy docker kubernetes and ArgoCD

<div align="center">

#  Blood Bank Application — End-to-End DevOps Pipeline

</div>

---

##  What Is This Project?

This project is a **real-world, production-grade DevOps implementation** of a PHP-based Blood Bank Management System.

It demonstrates the **complete software delivery lifecycle** — from a developer pushing code, through automated quality checks and security scanning, to a zero-touch Kubernetes deployment via GitOps.

>  **Built to simulate how modern engineering teams ship software at scale.**

---

##  Pipeline Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Developer  │────▶│   GitHub    │────▶│   Jenkins   │────▶│  SonarQube  │
│  (Code Push) │     │  (Source)   │     │  (CI/CD)    │     │ (QA Gate)   │
└─────────────┘     └─────────────┘     └─────────────┘     └──────┬──────┘
                                                                     │ PASS
                                                              ┌──────▼──────┐
                                                              │    Trivy    │
                                                              │  (Security) │
                                                              └──────┬──────┘
                                                                     │
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
│  Kubernetes  │◀────│   ArgoCD    │◀────│  GitOps     │◀────│  DockerHub  │
│  Cluster    │     │  (Deploy)   │     │  Repo (Git) │     │  (Registry) │
└──────┬──────┘     └─────────────┘     └─────────────┘     └─────────────┘
       │
       ▼
  NGINX Ingress ──▶ End Users


```

---

## 🛠️ Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| **Source Control** | GitHub | Version control & GitOps source of truth |
| **CI/CD** | Jenkins | Pipeline automation |
| **Code Quality** | SonarQube | Static analysis & quality gates |
| **Security** | Trivy | Container vulnerability scanning |
| **Containerization** | Docker | App + DB image builds |
| **Registry** | DockerHub | Image storage & versioning |
| **Orchestration** | Kubernetes (kops) | Container scheduling & scaling |
| **GitOps** | ArgoCD | Declarative deployment & auto-sync |
| **Monitoring** | Prometheus + Metrics Server | Cluster observability |
| **Cloud** | AWS EC2 | Infrastructure provider |

---

##  Infrastructure Setup

### EC2 Instance
```
Instance Type : m7i-flex.large
OS            : Amazon Linux
IAM Role      : Admin Access
Ports Open    : 8080 (Jenkins), 9000 (SonarQube), 80/443 (App)
```

### Tools Installed

```bash
# Java (Jenkins dependency)
yum install java-17-amazon-corretto -y

# Docker
yum install git docker -y
systemctl start docker

# Jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/rpm-stable/jenkins.repo
yum install jenkins -y
systemctl start jenkins

# SonarQube (containerised)
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

# Trivy (container scanner)
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sudo sh -s -- -b /usr/local/bin v0.69.3
```

---

##  CI/CD Pipeline — Step by Step

```
Stage 1: Code Checkout       → Pull from GitHub
Stage 2: SonarQube Analysis  → Scan for bugs, code smells, vulnerabilities
Stage 3: Quality Gate        → Block pipeline if quality thresholds not met
Stage 4: Docker Build        → Build app + database images
Stage 5: Trivy Scan          → Scan images for CVEs before pushing
Stage 6: DockerHub Push      → Tag & push verified images
Stage 7: GitOps Update       → Update Kubernetes manifest with new image tag
Stage 8: ArgoCD Sync         → Automatically detect and deploy changes

##  Docker

### Image Builds
```bash
# Application image
docker build -t appimage .

# Database image
docker build -t dbimage ./database
```

### Tag & Push to DockerHub
```bash
docker tag appimage sirisharokkam/bloodbank:appimage
docker tag dbimage  sirisharokkam/bloodbank:dbimage

docker push sirisharokkam/bloodbank:appimage
docker push sirisharokkam/bloodbank:dbimage
```

---

## ☸️ Kubernetes — Cluster Setup (kops on AWS)

### Install kops + kubectl
```bash
# kops
curl -Lo kops https://github.com/kubernetes/kops/releases/download/v1.28.0/kops-linux-amd64
chmod +x kops && mv kops /usr/local/bin/

# kubectl
curl -LO https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl
chmod +x kubectl && mv kubectl /usr/local/bin/
```

### Create Cluster
```bash
export KOPS_STATE_STORE=s3://<your-state-bucket>

kops create cluster \
  --name siri.k8s.local \
  --zones us-east-1a,us-east-1b \
  --master-size m7i-flex.large \
  --node-count 3 \
  --node-size c7i-flex.large

kops update cluster --name siri.k8s.local --yes
```

---

## 📂 Kubernetes Manifests

| Manifest | Resource | Purpose |
|---|---|---|
| `app-deployment.yaml` | Deployment | Manages PHP app pods |
| `db-statefulset.yaml` | StatefulSet | Stable identity for MySQL |
| `app-service.yaml` | Service (ClusterIP) | Internal service discovery |
| `ingress.yaml` | Ingress | External HTTP routing via NGINX |
| `configmap.yaml` | ConfigMap | Non-sensitive app config |
| `secrets.yaml` | Secret (Opaque) | DB credentials, API keys |
| `hpa.yaml` | HPA | Auto-scales pods under load |
| `pdb.yaml` | PDB | Maintains uptime during rollouts |
| `resourcequota.yaml` | ResourceQuota | Limits namespace resource usage |


---

##  GitOps with ArgoCD

Instead of Jenkins deploying directly to Kubernetes:

```
Jenkins pipeline  ──▶  Updates image tag in Git (GitOps repo)
ArgoCD watches Git ──▶  Detects change ──▶  Deploys automatically
```

### Why GitOps?
-  **Declarative** — desired state is always in Git
-  **Self-healing** — ArgoCD reverts manual changes automatically
-  **Auditable** — every deployment is a Git commit
-  **Rollback** = a `git revert`

> 🔒 In a production scenario, this would be integrated with **AWS Secrets Manager** or **HashiCorp Vault**.

---

##  Monitoring

```bash
# Deploy Metrics Server (required for HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify HPA is receiving metrics
kubectl top pods -n bloodbank

# Check cluster node resource usage
kubectl top nodes
```

**Prometheus** is deployed via Helm for deeper cluster observability, scraping metrics from all pods and nodes.

##  Troubleshooting Guide or Faced issues

| Symptom | Root Cause | Fix |
|---|---|---|
| `ImagePullBackOff` | Wrong image name or tag | Verify DockerHub image exists; check secret for private repos |
| `CrashLoopBackOff` | App misconfiguration | `kubectl logs <pod>` to inspect startup errors |
| StatefulSet pod stuck | PVC in failed state | Delete PVC: `kubectl delete pvc <name>`, recreate |
| Ingress returning 404 | Ingress controller not installed | `kubectl get pods -n ingress-nginx` |
| HPA not scaling | Metrics server missing | Verify: `kubectl top pods` returns data |
| ArgoCD out of sync | Webhook not configured | Manual sync: `argocd app sync bloodbank` |


##  Project Highlights

| Feature | Implementation |
|---|---|
| Zero-downtime deployments | Rolling updates via Kubernetes Deployment |
| Auto-scaling | HPA scales pods based on CPU utilization |
| Fault tolerance | PDB ensures minimum pods always available |
| Security scanning | Trivy blocks pipeline on HIGH/CRITICAL CVEs |
| GitOps pattern | ArgoCD syncs from Git — no manual `kubectl apply` |
| Self-healing cluster | ArgoCD detects and reverts configuration drift |



## 👩‍💻 Author

**Sirisha Rokkam**  
DevOps Engineer

---

<div align="center">

*This project is open source application*

</div>

