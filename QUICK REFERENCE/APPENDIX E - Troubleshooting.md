# APPENDIX E — Troubleshooting Guide
## DevSecOps Lab Book — Problem Solving Reference

---

## Common Issues & Solutions

### Kubernetes Issues

**Problem: Pods stuck in "Pending" state**
```bash
# Check events
kubectl describe pod pod-name -n namespace

# Common causes:
1. Insufficient resources: kubectl top nodes
2. Node affinity: Check nodeSelector/tolerations
3. PVC not bound: kubectl get pvc
4. Service account missing: kubectl get sa -n namespace

# Solution:
kubectl delete pod pod-name -n namespace  # Recreate pod
kubectl scale deployment app --replicas=2 -n namespace
```

**Problem: DNS resolution failing in pods**
```bash
# Test DNS from pod
kubectl exec -it pod-name -- nslookup kubernetes.default

# Check CoreDNS
kubectl get pods -n kube-system | grep coredns

# Verify /etc/resolv.conf inside pod
kubectl exec -it pod-name -- cat /etc/resolv.conf

# Solution:
kubectl scale deployment coredns --replicas=3 -n kube-system
kubectl restart pod coredns-xxx -n kube-system
```

**Problem: CrashLoopBackOff pod state**
```bash
# Check logs
kubectl logs pod-name -n namespace --previous  # Previous run
kubectl logs pod-name -n namespace              # Current attempt

# Common causes:
1. Application crashes: Check logs for errors
2. Missing dependencies: Check image contents
3. Resource limits: Increase CPU/memory

# Debug:
kubectl describe pod pod-name -n namespace
kubectl get events -n namespace --sort-by='.lastTimestamp'
```

### Security Issues

**Problem: Certificate errors (SSL/TLS)**
```bash
# Check certificate
openssl x509 -in cert.pem -text -noout

# Verify chain
openssl verify -CAfile ca.pem cert.pem

# Check expiration
openssl x509 -in cert.pem -noout -dates

# Solution:
# Regenerate certificate, update in Vault/Kubernetes
vault write pki_int/issue/lab common_name=example.com
kubectl create secret tls tls-secret --cert=cert.pem --key=key.pem
```

**Problem: Authentication failures**
```bash
# Check credentials
kubectl config view
kubectl auth can-i get pods

# Verify OIDC
curl -k https://auth.devsecops.com/.well-known/openid-configuration

# Check Teleport
tsh status

# Solution:
kubectl config set-credentials user --token=new-token
tsh login --proxy=teleport.devsecops.com
```

### Network Issues

**Problem: Service not accessible**
```bash
# Check service
kubectl get svc -n namespace
kubectl describe svc service-name -n namespace

# Check endpoints
kubectl get endpoints -n namespace

# Test connectivity
kubectl run -it --rm debug --image=busybox -- sh
# Inside pod: wget service-name:port

# Check firewall
ufw status | grep port-number
netstat -tulpn | grep port-number

# Solution:
# Verify selector matches pod labels
# Check service port vs container port
# Verify NetworkPolicy allows traffic
```

**Problem: DNS not resolving**
```bash
# Check DNS server
cat /etc/resolv.conf

# Query DNS server
dig @10.0.2.10 example.com

# Check BIND logs
tail -f /var/log/bind/named.log

# Solution:
# Restart BIND: systemctl restart named
# Verify zones: named-checkzone example.com /etc/bind/zones/db.example.com
# Check ACLs in named.conf
```

### Performance Issues

**Problem: High CPU/Memory usage**
```bash
# Monitor metrics
kubectl top nodes
kubectl top pods -A

# Check for memory leaks
kubectl get pods -A --sort-by=.spec.containers[0].resources.requests.memory

# Profile application
kubectl port-forward pod-name 6060:6060
curl http://localhost:6060/debug/pprof

# Solution:
# Increase resource requests/limits
# Optimize application code
# Scale horizontally
```

---

*DevSecOps Lab Book — Appendix E: Troubleshooting*

