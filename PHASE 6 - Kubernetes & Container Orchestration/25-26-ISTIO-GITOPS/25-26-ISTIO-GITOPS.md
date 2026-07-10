# Chapters 25-26 — Advanced Kubernetes: Istio & GitOps

## DevSecOps Lab Book — Phase 6: Container Orchestration (Advanced)

---

# Chapter 25 — Istio: Service Mesh & Advanced Networking

---

> **Chapter Goal**
> 
> Deploy Istio service mesh for advanced traffic management, security, and observability across services.

---

## 25.1 What is a Service Mesh?

### Problem: Service-to-Service Communication

```
Before Istio:
  Service A → Service B
  
  Problems:
  ❌ Retry logic in application code
  ❌ Timeouts configured in app
  ❌ Load balancing in app
  ❌ Security (mTLS) in app
  ❌ Circuit breakers in app
  ❌ Observability scattered
```

### Solution: Istio Service Mesh

```
With Istio:
  Service A → Envoy Proxy → Envoy Proxy → Service B
  
  Mesh handles:
  ✅ Retries (automatic)
  ✅ Timeouts (automatic)
  ✅ Load balancing (sophisticated)
  ✅ mTLS (automatic encryption)
  ✅ Circuit breakers (automatic)
  ✅ Observability (Prometheus + Grafana)
  
  Applications remain simple!
```

---

## 25.2 Istio Installation

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.15.0
export PATH=$PWD/bin:$PATH

# Install Istio
istioctl install --set profile=demo -y

# Label namespace for injection
kubectl label namespace default istio-injection=enabled

# Verify installation
kubectl get all -n istio-system
```

---

## 25.3 Traffic Management

### VirtualService & DestinationRule

```yaml
# Virtual Service: Define how traffic is routed
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - myapp
  http:
  - match:
    - uri:
      prefix: "/api/v1"
    route:
    - destination:
        host: myapp
        port:
          number: 8080
        subset: v1
      weight: 80  # 80% to v1
    - destination:
        host: myapp
        port:
          number: 8080
        subset: v2
      weight: 20  # 20% to v2 (canary)
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 2s

---
# Destination Rule: Define subsets (versions)
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-dr
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

### Canary Deployment

```bash
# Deploy v1 (current)
kubectl apply -f myapp-v1.yaml

# Deploy v2 (canary)
kubectl apply -f myapp-v2.yaml

# Apply VirtualService (80% v1, 20% v2)
kubectl apply -f virtualservice.yaml

# Monitor metrics
# If v2 error rate is good, increase traffic
# 50% → 100%
```

---

## 25.4 Security: mTLS

### Enable mTLS (Mutual TLS)

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT  # Enforce mTLS for all traffic
```

Result:

- ✅ All service-to-service communication encrypted
- ✅ Mutual authentication (each service verifies the other)
- ✅ Certificate management automatic (Istio handles rotation)
- ✅ Transparent to applications

---

## 25.5 Observability

### Prometheus Metrics

```bash
# Istio automatically exports metrics:
# - Request rate (requests/second)
# - Error rate (errors/second)
# - Latency (p50, p90, p99)

# Query in Prometheus:
rate(istio_requests_total[5m])
histogram_quantile(0.99, rate(istio_request_duration_milliseconds_bucket[5m]))
```

### Kiali Visualization

```bash
# Istio installs Kiali (service mesh visualization)
kubectl port-forward -n istio-system svc/kiali 20000:20000

# Visit: http://localhost:20000
# See: Service mesh topology, traffic flow, error rates
```

---

# Chapter 26 — GitOps: Automated Deployments with ArgoCD

---

> **Chapter Goal**
> 
> Implement GitOps: Git as source of truth, ArgoCD for continuous deployment. All Kubernetes changes through Git commits.

---

## 26.1 GitOps Principles

### The GitOps Flow

```
Developer
    ↓
Commit code to Git
    ↓
CI pipeline tests & builds (Phase 4)
    ↓
Push image to Harbor
    ↓
Update Kubernetes manifests in Git repo
    ↓
ArgoCD detects Git change
    ↓
Automatically deploys to Kubernetes
    ↓
Desired state (Git) = Actual state (Kubernetes)
```

---

## 26.2 ArgoCD Installation

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward -n argocd svc/argocd-server 8080:443

# Get initial password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Visit: https://localhost:8080
# Login: admin / <password>
```

---

## 26.3 Create ArgoCD Application

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://gitlab.devsecops.com/root/myapp-helm.git
    targetRevision: main
    path: helm/myapp
    
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
    
  syncPolicy:
    automated:
      prune: true      # Delete if removed from Git
      selfHeal: true   # Resync if drift detected
    syncOptions:
    - CreateNamespace=true
```

Deploy:

```bash
kubectl apply -f argocd-application.yaml

# Check sync status
argocd app get myapp
argocd app sync myapp  # Manual sync if needed
```

---

## 26.4 Git Repository Structure

```
myapp-helm/
├── helm/
│   ├── myapp/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── values-dev.yaml
│   │   ├── values-prod.yaml
│   │   └── templates/
│   └── postgresql/
└── argocd/
    ├── apps/
    │   ├── myapp-dev.yaml
    │   ├── myapp-prod.yaml
    │   └── postgresql.yaml
    └── projects/
        └── default.yaml
```

---

## 26.5 Complete GitOps Workflow

### Update Application Version

```bash
# 1. Developer pushes new image to Harbor (Phase 4)
# 2. CI pipeline creates new Git tag

# 3. Update Helm values
git clone https://gitlab.devsecops.com/root/myapp-helm.git
cd myapp-helm
vim helm/myapp/values.yaml
# Change: tag: "1.0.0" → tag: "1.1.0"

# 4. Commit & push
git add helm/myapp/values.yaml
git commit -m "Update myapp to v1.1.0"
git push origin main

# 5. ArgoCD automatically detects change and deploys
# No manual kubectl apply needed!

# 6. Verify deployment
kubectl get pods -n myapp
# Should see new version running
```

---

## 26.6 Multi-Environment Setup

### Dev Environment

```yaml
# argocd/apps/myapp-dev.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  source:
    repoURL: https://gitlab.devsecops.com/root/myapp-helm.git
    path: helm/myapp
    helm:
      releaseName: myapp
      values: |
        replicaCount: 1
        image:
          tag: latest
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp-dev
```

### Prod Environment

```yaml
# argocd/apps/myapp-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  source:
    repoURL: https://gitlab.devsecops.com/root/myapp-helm.git
    path: helm/myapp
    helm:
      releaseName: myapp
      values: |
        replicaCount: 5
        image:
          tag: "1.1.0"  # Pinned version
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp-prod
```

---

## 26.7 Integration with CI/CD (Phase 4)

### GitLab CI → ArgoCD

```yaml
# .gitlab-ci.yml - Update Helm values after push
deploy:
  stage: deploy
  script:
    - |
      # Clone Helm repo
      git clone https://gitlab.devsecops.com/root/myapp-helm.git
      cd myapp-helm
      
      # Update image tag
      sed -i "s/tag: .*/tag: \"$CI_COMMIT_SHA\"/" helm/myapp/values.yaml
      
      # Commit & push (ArgoCD syncs automatically)
      git config user.email "ci@devsecops.com"
      git config user.name "CI Bot"
      git add helm/myapp/values.yaml
      git commit -m "Update image to $CI_COMMIT_SHA"
      git push https://oauth2:$GITLAB_TOKEN@gitlab.devsecops.com/root/myapp-helm.git main
```

---

## 26.8 Rollback & Recovery

### Automatic Rollback on Failure

```bash
# ArgoCD can automatically rollback if health checks fail

# Define health checks in ArgoCD Application
spec:
  project: default
  source:
    ...
  destination:
    ...
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      # Auto-rollback on failed sync
```

### Manual Rollback

```bash
# Check sync history
argocd app history myapp

# Rollback to previous sync
argocd app rollback myapp 2

# Result: Git is reverted, ArgoCD deploys old version
```

---

## 26.9 Chapter Summary

|Tool|Purpose|
|---|---|
|**Istio**|Service mesh (traffic, security, observability)|
|**ArgoCD**|GitOps deployment automation|
|**Git**|Source of truth|
|**Kubernetes**|Actual deployed state|

💡 **Key Takeaway — Git-Driven Infrastructure:**

Before GitOps:

- ❌ Manual kubectl apply
- ❌ Drift between Git and cluster
- ❌ Hard to track changes
- ❌ Difficult rollbacks

After GitOps (ArgoCD):

- ✅ All changes through Git commits
- ✅ Git = source of truth
- ✅ Auto-sync to cluster
- ✅ Easy rollbacks (just revert commit)
- ✅ Audit trail in Git

---

_DevSecOps Lab Book — Zero to Hero | Chapters 25-26 | Phase 6: Advanced_