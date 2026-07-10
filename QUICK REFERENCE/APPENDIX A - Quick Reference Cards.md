# APPENDIX A — Quick Reference Cards & Cheat Sheets
## DevSecOps Lab Book — One-Page Guides for Each Phase

---

## PHASE 1: FOUNDATION (One-Page Cheat Sheet)

```
BIND9 DNS Setup:
├─ Install: apt-get install bind9
├─ Config: /etc/bind/named.conf
├─ Split-brain: internal vs external zones
└─ Test: dig @10.0.2.10 devsecops.com

LDAP/Authentik Identity:
├─ LDAP: ldap://10.0.2.11:389, base DN: dc=devsecops,dc=com
├─ OIDC: http://auth.devsecops.com:9000
├─ Create user: ldapadd -D "cn=admin,dc=devsecops,dc=com" < user.ldif
└─ Test: ldapsearch -x -H ldap://10.0.2.11 -b dc=devsecops,dc=com

HAProxy Load Balancer:
├─ Config: /etc/haproxy/haproxy.cfg
├─ Frontend: bind *:443, default_backend backend
├─ Backend: balance roundrobin, server cp-01 10.0.2.20:6443
└─ Test: curl -k https://10.0.2.12

HTTPS/TLS Certificate:
├─ Generate: openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem
├─ Verify: openssl x509 -in cert.pem -text -noout
├─ Authority: /etc/ssl/certs/ca-certificates.crt
└─ HAProxy: cat cert.key cert.crt > /etc/haproxy/certs/devsecops.com.pem
```

## PHASE 2: AUTOMATION (One-Page Cheat Sheet)

```
Packer Image Building:
├─ Template: ubuntu-22.04.pkr.hcl
├─ Provisioners: provisioners.sh (install tools)
├─ Build: packer build ubuntu-22.04.pkr.hcl
└─ Output: /var/lib/libvirt/images/ubuntu-22.04-packer.qcow2

Terraform IaC:
├─ Init: terraform init
├─ Plan: terraform plan -out=tfplan
├─ Apply: terraform apply tfplan
├─ Destroy: terraform destroy
└─ State: terraform show | terraform state list

Ansible Playbooks:
├─ Inventory: inventory.ini (host list)
├─ Playbook: site.yml (configuration)
├─ Run: ansible-playbook -i inventory.ini site.yml
├─ Check: ansible all -m ping
└─ Vars: group_vars/, host_vars/
```

## PHASE 3: SECURITY (One-Page Cheat Sheet)

```
Vault Secrets:
├─ Init: vault operator init
├─ Unseal: vault operator unseal (3x with keys)
├─ Auth: vault login (with token)
├─ Write: vault kv put secret/myapp password=secret
├─ Read: vault kv get secret/myapp
└─ Admin: vault audit enable file file_path=/var/log/vault/audit.log

Teleport Access:
├─ Login: tsh login --proxy=teleport.devsecops.com:3023
├─ SSH: tsh ssh ubuntu@cp-01
├─ Record: All sessions recorded in /var/lib/teleport
└─ Audit: tctl sessions ls

PKI Certificates:
├─ Generate: vault write -type=pem pki_int/issue/lab common_name=example.com
├─ Renew: vault write -f pki_int/rotate/crl
├─ Revoke: vault write pki_int/revoke serial_number=xxx
└─ CRL: vault write -f pki_int/tidy
```

## PHASE 4: CI/CD (One-Page Cheat Sheet)

```
GitLab Pipelines:
├─ Config: .gitlab-ci.yml
├─ Stages: build → test → deploy
├─ Trigger: Push to repo
├─ Access: https://gitlab.devsecops.com
└─ Runner: gitlab-runner install & start

SonarQube SAST:
├─ Scan: sonar-scanner -Dsonar.projectKey=myapp
├─ Dashboard: https://sonarqube.devsecops.com
├─ Quality Gate: > 80% coverage, 0 critical issues
└─ Integration: GitLab CI → SonarQube

Harbor Registry:
├─ Login: docker login harbor.devsecops.com
├─ Push: docker push harbor.devsecops.com/myapp:latest
├─ Scan: trivy image harbor.devsecops.com/myapp:latest
└─ Policy: Block images with critical vulnerabilities
```

## PHASE 5: ADVANCED (One-Page Cheat Sheet)

```
High Availability:
├─ Vault HA: Consul backend, 3+ nodes
├─ PostgreSQL: Patroni auto-failover
├─ Redis: Sentinel with 3+ nodes
└─ VIP: Keepalived at 10.0.2.100

Disaster Recovery:
├─ Backup frequency: Hourly snapshots
├─ Retention: 30 days
├─ Test recovery: Monthly
└─ RTO: < 1 hour, RPO: < 15 minutes

Performance:
├─ pgBouncer connection pooling
├─ Redis multi-layer caching
├─ Kernel tuning (sysctl)
└─ Load testing (JMeter)
```

## PHASE 6: KUBERNETES (One-Page Cheat Sheet)

```
Kubernetes Cluster:
├─ Init: kubeadm init --control-plane-endpoint=10.0.2.25:6443
├─ Join CP: kubeadm join --control-plane --certificate-key xxx
├─ Join Worker: kubeadm join --token xxx
├─ CNI: kubectl apply -f calico.yaml
└─ Verify: kubectl get nodes

Helm Charts:
├─ Create: helm create myapp
├─ Install: helm install myapp ./myapp-chart
├─ Upgrade: helm upgrade myapp ./myapp-chart
└─ Rollback: helm rollback myapp

Istio Service Mesh:
├─ Install: istioctl install --set profile=demo
├─ VirtualService: Route traffic between versions
├─ DestinationRule: Define service subsets
└─ PeerAuth: Enable mTLS

GitOps:
├─ Install ArgoCD: kubectl apply -n argocd -f install.yaml
├─ Create App: argocd app create myapp --repo https://...
├─ Sync: argocd app sync myapp
└─ Rollback: argocd app rollback myapp 2
```

---

*DevSecOps Lab Book — Appendix A: Quick Reference*

