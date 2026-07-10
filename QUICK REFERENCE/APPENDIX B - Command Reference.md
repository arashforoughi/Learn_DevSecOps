# APPENDIX B — Command Reference Guide
## DevSecOps Lab Book — Essential Commands for Each Phase

---

## System Administration

```bash
# Network troubleshooting
ping 10.0.2.10                          # Test connectivity
traceroute 10.0.2.10                    # Trace route
netstat -tulpn                          # Show listening ports
ss -tunap                               # More modern netstat
ip route show                           # Show routing table
iptables -L -n                          # Show firewall rules

# DNS queries
nslookup devsecops.com                  # Simple lookup
dig devsecops.com +trace                # Trace DNS
host devsecops.com 10.0.2.10            # Query specific server

# Service management
systemctl status sshd                   # Service status
systemctl enable sshd                   # Auto-start
systemctl restart sshd                  # Restart service
journalctl -u sshd -f                   # Follow logs
```

## Kubernetes Commands

```bash
# Cluster information
kubectl cluster-info                    # Cluster details
kubectl get nodes -o wide              # Node status
kubectl describe node cp-01            # Node details
kubectl get events -A --sort-by='.lastTimestamp'  # All events

# Pod management
kubectl get pods -A                     # All pods
kubectl describe pod pod-name -n ns     # Pod details
kubectl logs pod-name -n ns            # Container logs
kubectl exec -it pod-name -n ns -- /bin/bash  # Shell access

# Deployments
kubectl get deployments -A              # All deployments
kubectl rollout status deployment/app  # Deployment status
kubectl rollout history deployment/app # History
kubectl rollout undo deployment/app     # Rollback

# Kubernetes API
kubectl api-resources                   # Available resources
kubectl explain deployment              # Resource documentation
kubectl get deployment -o yaml          # YAML output
```

## Security Commands

```bash
# SSL/TLS
openssl x509 -in cert.pem -text -noout        # View certificate
openssl s_client -connect host:443 </dev/null # Test HTTPS
openssl verify -CAfile ca.pem cert.pem       # Verify chain

# SSH
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519   # Generate key
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host  # Deploy key
ssh -v user@host                              # Debug SSH

# Firewall
ufw status numbered                     # Show firewall rules
ufw allow 22/tcp                       # Allow SSH
ufw delete 1                           # Delete rule by number

# Audit
ausearch -k passwd_changes             # Search audit logs
ausearch -ts recent                    # Recent events
aureport --user-summary                # User summary
```

## Container Commands

```bash
# Docker
docker run -it ubuntu:22.04 /bin/bash         # Run container
docker ps -a                                   # List containers
docker images                                  # List images
docker build -t myapp:latest .               # Build image
docker push harbor.devsecops.com/myapp:latest # Push image

# Container inspection
docker inspect container-id             # Container details
docker logs container-id                # Container logs
docker exec -it container-id /bin/bash  # Execute in container

# Trivy scanning
trivy image harbor.devsecops.com/app:latest  # Scan image
trivy config /path/to/config.yaml      # Scan config
trivy fs /path/to/filesystem           # Scan filesystem
```

## Git Commands

```bash
# Repository
git clone https://url.git               # Clone repo
git status                              # Status
git log --oneline -10                  # Recent commits
git diff HEAD                          # Show changes

# Branching
git branch feature/new-feature          # Create branch
git checkout feature/new-feature       # Switch branch
git merge feature/new-feature          # Merge branch

# Commits
git add .                              # Stage changes
git commit -m "message"                # Commit changes
git push origin main                   # Push to remote
git pull origin main                   # Pull from remote
```

## Ansible Commands

```bash
# Inventory & testing
ansible all -i inventory.ini -m ping        # Test connectivity
ansible all -i inventory.ini -m setup       # Gather facts
ansible-inventory -i inventory.ini --list   # List inventory

# Playbooks
ansible-playbook -i inventory.ini site.yml           # Run playbook
ansible-playbook -i inventory.ini site.yml -v       # Verbose
ansible-playbook -i inventory.ini site.yml --check  # Dry-run

# Modules
ansible all -m shell -a "echo hello"        # Run command
ansible all -m copy -a "src=file dest=/tmp" # Copy file
ansible all -m service -a "name=ssh state=started" # Service
```

---

*DevSecOps Lab Book — Appendix B: Command Reference*

