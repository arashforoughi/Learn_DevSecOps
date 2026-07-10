# APPENDIX D — Configuration & Automation Templates
## DevSecOps Lab Book — Ready-to-Use Templates

---

## Kubernetes Pod Security Template

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: production
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 3000
    seccompProfile:
      type: RuntimeDefault

  serviceAccountName: app-sa

  containers:
  - name: app
    image: harbor.devsecops.com/myapp:latest
    imagePullPolicy: Always

    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]

    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 256Mi

    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5

    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: config
      mountPath: /etc/app
      readOnly: true

  volumes:
  - name: tmp
    emptyDir:
      medium: Memory
      sizeLimit: 512Mi
  - name: config
    configMap:
      name: app-config
      defaultMode: 0444
```

## Terraform Module Template

```hcl
# modules/kubernetes_cluster/main.tf

variable "cluster_name" {
  type = string
}

variable "node_count" {
  type = number
  default = 3
}

variable "machine_type" {
  type = string
  default = "n2-standard-4"
}

resource "kubernetes_cluster" "main" {
  name = var.cluster_name
  
  node_pool {
    name = "default"
    node_count = var.node_count
    
    node_config {
      machine_type = var.machine_type
      disk_size_gb = 100
    }
  }
}

output "cluster_endpoint" {
  value = kubernetes_cluster.main.endpoint
}

output "cluster_ca_certificate" {
  value = base64decode(kubernetes_cluster.main.master_auth[0].cluster_ca_certificate)
}
```

## Ansible Playbook Template

```yaml
---
- name: Deploy application
  hosts: kubernetes_masters
  vars:
    app_name: myapp
    app_version: "1.0.0"
    container_registry: "harbor.devsecops.com"

  tasks:
  - name: Create namespace
    kubernetes.core.k8s:
      name: "{{ app_name }}"
      api_version: v1
      kind: Namespace
      state: present

  - name: Create secret for registry
    kubernetes.core.k8s:
      state: present
      definition:
        apiVersion: v1
        kind: Secret
        metadata:
          name: regcred
          namespace: "{{ app_name }}"
        type: kubernetes.io/dockercfg
        data:
          .dockercfg: "{{ dockerconfig | b64encode }}"

  - name: Deploy application
    kubernetes.core.helm:
      name: "{{ app_name }}"
      chart_ref: "{{ container_registry }}/{{ app_name }}"
      release_namespace: "{{ app_name }}"
      values:
        image:
          repository: "{{ container_registry }}/{{ app_name }}"
          tag: "{{ app_version }}"

  - name: Verify deployment
    kubernetes.core.k8s_info:
      kind: Deployment
      name: "{{ app_name }}"
      namespace: "{{ app_name }}"
    register: deployment
    until: deployment.resources[0].status.readyReplicas == deployment.resources[0].spec.replicas
    retries: 10
    delay: 30
```

## GitLab CI/CD Pipeline Template

```yaml
stages:
  - build
  - test
  - security
  - deploy

variables:
  REGISTRY: harbor.devsecops.com
  IMAGE_TAG: ${CI_COMMIT_SHORT_SHA}

build:
  stage: build
  image: docker:latest
  script:
    - docker build -t ${REGISTRY}/myapp:${IMAGE_TAG} .
    - docker push ${REGISTRY}/myapp:${IMAGE_TAG}

test:
  stage: test
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pytest --cov=app tests/

sast:
  stage: security
  image: sonarqube-cli:latest
  script:
    - sonar-scanner -Dsonar.projectKey=myapp

dependency-check:
  stage: security
  script:
    - dependency-check.sh --project myapp

deploy:
  stage: deploy
  image: alpine/helm:latest
  script:
    - helm upgrade myapp ./chart --install
  only:
    - main
```

## Firewall Rules Template

```bash
#!/bin/bash
# configure-firewall.sh

# Reset
ufw reset

# Default policies
ufw default deny incoming
ufw default allow outgoing
ufw default deny routed

# Essential services
ufw allow 22/tcp           # SSH
ufw allow 80/tcp           # HTTP
ufw allow 443/tcp          # HTTPS

# Kubernetes
ufw allow 6443/tcp         # API server
ufw allow 10250/tcp        # Kubelet

# Application ports
ufw allow from 10.0.0.0/8 to any port 5432  # PostgreSQL

# Rate limiting
ufw limit 22/tcp           # SSH brute force protection

# DNS (if needed)
ufw allow 53/tcp
ufw allow 53/udp

# Enable
ufw enable
```

---

*DevSecOps Lab Book — Appendix D: Templates*

