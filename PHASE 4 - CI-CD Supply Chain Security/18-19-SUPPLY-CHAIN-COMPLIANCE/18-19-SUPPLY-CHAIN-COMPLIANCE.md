# Chapters 18-19 — Supply Chain Security & Compliance

## DevSecOps Lab Book — Phase 4: CI/CD Supply Chain (Advanced)

---

# Chapter 18 — Supply Chain Security: SLSA, Attestation & Signing

---

> **Chapter Goal**
> 
> Implement SLSA (Supply-chain Levels for Software Artifacts) framework. Ensure artifacts are properly built, signed, and attested throughout the pipeline.
> 
> **SLSA Levels:**
> 
> - Level 1: Scripted builds (what we have now)
> - Level 2: Version control + CI/CD (what we're implementing)
> - Level 3: Signed provenance + hardened CI/CD
> - Level 4: Fully hermetic builds

---

## 18.1 Image Signing & Verification

### Enable Docker Content Trust

```bash
# In GitLab CI/CD:
export DOCKER_CONTENT_TRUST=1
export DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io

# Build and sign image
docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .

# Push signed image (requires passphrase)
docker push $IMAGE_NAME:$CI_COMMIT_SHA
# Prompts for delegation key passphrase

# Client verifies signature on pull
docker pull $IMAGE_NAME:$CI_COMMIT_SHA
```

### In Harbor

```bash
# Enable content trust per project:
# Projects → myapp → Enable content trust

# Mandate signature verification:
# Administration → System Settings
# - Require image signature before push
- Block images without valid signatures
```

---

## 18.2 Build Provenance & Attestation

Generate attestations for every build:

```yaml
# In .gitlab-ci.yml
generate_provenance:
  stage: build_image
  script:
    - |
      # Create SLSA provenance attestation
      cat > provenance.json << EOF
      {
        "version": "0.0.1",
        "artifact": {
          "uri": "$IMAGE_NAME:$CI_COMMIT_SHA",
          "digest": "$(docker inspect $IMAGE_NAME:$CI_COMMIT_SHA | jq -r '.[0].Id')"
        },
        "builder": {
          "id": "https://gitlab.devsecops.com"
        },
        "buildConfig": {
          "source": "$CI_PROJECT_URL",
          "entrypoint": ".gitlab-ci.yml"
        },
        "metadata": {
          "buildInvocationId": "$CI_PIPELINE_ID",
          "buildStartTime": "$(date -u +'%Y-%m-%dT%H:%M:%SZ')",
          "completeness": {
            "parameters": false,
            "environment": false,
            "materials": false
          },
          "reproducible": false
        },
        "materials": [
          {
            "uri": "$CI_PROJECT_URL/-/commit/$CI_COMMIT_SHA",
            "digest": "$CI_COMMIT_SHA"
          }
        ]
      }
      EOF
      
      # Sign attestation
      gpg --armor --detach-sign provenance.json
      
      # Upload to artifact store
      curl -X POST \
        -H "Authorization: Bearer $ATTESTATION_TOKEN" \
        -F "provenance=@provenance.json" \
        -F "signature=@provenance.json.asc" \
        https://attestation.devsecops.com/api/v1/attestations
```

---

## 18.3 Supply Chain Levels (SLSA)

Track SLSA compliance:

```
SLSA Level 1 ✅ (What we have)
├─ Scripted builds
├─ Build output retention
└─ Build process documentation

SLSA Level 2 ✅ (Implementing)
├─ Version controlled source
├─ Tracked, automated builds
├─ Cryptographic provenance
└─ Access control

SLSA Level 3 (Future)
├─ Signed, reviewed commits
├─ Hardened CI/CD
├─ Verified dependencies
└─ Hermetic builds

SLSA Level 4 (Advanced)
├─ Fully hermetic builds
├─ Bit-for-bit reproducible
├─ Offline signing
└─ Auditability
```

---

## 18.4 Dependency Management

Track all dependencies in Software Bill of Materials (SBOM):

```bash
# Generate SBOM from Docker image
syft $IMAGE_NAME:$CI_COMMIT_SHA -o json > sbom.json
syft $IMAGE_NAME:$CI_COMMIT_SHA -o spdx > sbom.spdx

# Contents show all:
# ✅ Base OS packages (e.g., openssl 1.1.1)
# ✅ Python packages (Flask, requests, etc.)
# ✅ System libraries (libc, libssl, etc.)

# Example SBOM entry:
{
  "name": "openssl",
  "version": "1.1.1q",
  "type": "rpm",
  "purl": "pkg:rpm/openssl@1.1.1q?arch=x86_64",
  "licenses": ["OpenSSL"]
}
```

---

## 18.5 Vulnerability Attestation

Attest that image has been scanned for vulnerabilities:

```json
{
  "type": "vulnerability-scan",
  "timestamp": "2025-01-15T14:30:00Z",
  "image": "harbor.devsecops.com/myapp/myapp:v1.0",
  "scanner": "Trivy",
  "scanResult": {
    "critical": 0,
    "high": 0,
    "medium": 2,
    "low": 5
  },
  "passed": true,
  "signed": true,
  "signature": "-----BEGIN PGP SIGNATURE-----..."
}
```

---

# Chapter 19 — Compliance & Audit: SBOM, Reporting & Frameworks

---

> **Chapter Goal**
> 
> Implement compliance requirements (SOC2, PCI-DSS, HIPAA) through automated evidence collection and reporting.

---

## 19.1 Compliance Frameworks

Map DevSecOps to compliance requirements:

### SOC2 Type II

**Control: CC7.1 Restrict System Access**

- ✅ OIDC authentication (Authentik)
- ✅ RBAC in all tools
- ✅ Session recording (Teleport)
- ✅ Audit logs (all phases)

**Control: CC7.2 Software Security**

- ✅ Code review (GitLab merge requests)
- ✅ SAST scanning (SonarQube)
- ✅ Container scanning (Harbor/Trivy)
- ✅ Dependency scanning

**Control: CC9.2 Configuration**

- ✅ Infrastructure as code (OpenTofu)
- ✅ Version controlled configs
- ✅ Change auditing

### PCI-DSS

**Requirement 6: Secure Development**

- ✅ Code review process
- ✅ Static testing (SonarQube)
- ✅ Vulnerability remediation tracking (DefectDojo)

**Requirement 8: User Access**

- ✅ Authentication (Authentik OIDC)
- ✅ MFA (via Authentik)
- ✅ Access logging (Teleport)

---

## 19.2 Automated Compliance Evidence Collection

### Collect Evidence for Auditors

```bash
# Daily compliance evidence export
export_compliance_evidence() {
  REPORT_DATE=$(date +%Y-%m-%d)
  REPORT_DIR=/var/reports/compliance/$REPORT_DATE
  
  mkdir -p $REPORT_DIR
  
  # GitLab: Merge requests (code review evidence)
  curl -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
    https://gitlab.devsecops.com/api/v4/projects/1/merge_requests \
    > $REPORT_DIR/gitlab-merge-requests.json
  
  # SonarQube: Code quality metrics
  curl -H "Authorization: Bearer $SONARQUBE_TOKEN" \
    $SONARQUBE_URL/api/v2/issues \
    > $REPORT_DIR/sonarqube-issues.json
  
  # Harbor: Image scan results
  curl -H "Authorization: Bearer $HARBOR_TOKEN" \
    https://harbor.devsecops.com/api/v2/scan_results \
    > $REPORT_DIR/harbor-scan-results.json
  
  # Vault: Audit log
  curl -H "X-Vault-Token: $VAULT_TOKEN" \
    https://vault.devsecops.com/v1/sys/audit \
    > $REPORT_DIR/vault-audit.json
  
  # Teleport: Session recordings
  tctl audit get \
    --from=$(date -d '24 hours ago' +%Y-%m-%d) \
    --to=$REPORT_DATE \
    > $REPORT_DIR/teleport-audit.json
  
  # Package report
  tar -czf $REPORT_DIR/../compliance-$REPORT_DATE.tar.gz $REPORT_DIR
  echo "Compliance evidence collected: compliance-$REPORT_DATE.tar.gz"
}

# Schedule daily
0 2 * * * /usr/local/bin/export-compliance-evidence.sh
```

---

## 19.3 SBOM Generation & Distribution

Generate Software Bill of Materials for all artifacts:

```bash
# Generate SBOM at build time
docker_build_with_sbom() {
  # Build image
  docker build -t $IMAGE:$TAG .
  
  # Generate SBOM
  syft $IMAGE:$TAG -o cyclonedx > sbom.xml
  syft $IMAGE:$TAG -o spdx-json > sbom.spdx.json
  
  # Sign SBOM
  gpg --armor --detach-sign sbom.xml
  
  # Attach to image (as annotation)
  crane mutate $IMAGE:$TAG \
    --annotation "sbom=@sbom.spdx.json" \
    --annotation "sbom.signature=@sbom.xml.asc"
  
  # Publish SBOM
  curl -X POST \
    -H "Authorization: Bearer $SBOM_REPO_TOKEN" \
    -F "sbom=@sbom.spdx.json" \
    -F "image=$IMAGE:$TAG" \
    https://sbom-repo.devsecops.com/api/upload
}

# SBOM published and auditable
# Auditors can verify exactly what's in each image
```

---

## 19.4 Vulnerability Disclosure & SLA

Track vulnerability remediation per policy:

```yaml
# Vulnerability SLA Policy
Critical:     24 hours to fix or disable
High:         7 days to fix
Medium:       30 days to fix
Low:          90 days to fix

# DefectDojo tracks:
# - Discovery date
# - Due date (based on SLA)
# - Current status (Open/InProgress/Resolved)
# - Days remaining
# - Is overdue?

# Escalation:
# - 7 days before SLA: Notify team lead
# - At SLA: Escalate to manager
# - 7 days overdue: Executive escalation
```

---

## 19.5 Compliance Reporting

Generate compliance-ready reports:

```bash
# Annual Compliance Report
Compliance Report: DevSecOps Lab
Period: 2025-01-01 to 2025-12-31

SECURITY CONTROLS IMPLEMENTED:
✅ Access Control (CC7.1)
   - OIDC authentication: 100% coverage
   - Session recording: 100% audit trail
   - MFA enabled: Yes

✅ Software Security (CC7.2)
   - Code review rate: 95% (4,750/5,000 commits)
   - SAST scan rate: 100% (all commits)
   - Container scan rate: 100% (all images)
   - Critical findings fixed: 100% SLA compliance

✅ Configuration (CC9.2)
   - Infrastructure as code: 100%
   - Change audited: 100% (all commits)
   - Configuration drift detected: 0

VULNERABILITIES METRICS:
- Total findings: 245
- Critical fixed: 8/8 (100%)
- High fixed: 42/43 (98%)
- Overdue: 0
- Compliance: 100%

AUDIT TRAIL:
- Total events logged: 2.3M
- Retention period: 1 year
- Search capability: Real-time

CERTIFICATION:
This report certifies that the organization has implemented
and maintained security controls per SOC2/PCI-DSS/HIPAA
requirements for the period 2025-01-01 to 2025-12-31.

Signed by: CISO
Date: 2026-01-15
```

---

## 19.6 Continuous Compliance Monitoring

Monitor compliance continuously:

```yaml
# In Prometheus/Grafana
compliance_checks:
  - name: critical_vulnerabilities_overdue
    alert: "count(vulnerabilities.severity='critical' AND vulnerabilities.days_overdue>0) > 0"
    severity: critical
    
  - name: high_vulnerabilities_sla_breach
    alert: "count(vulnerabilities.severity='high' AND vulnerabilities.days_overdue>7) > 0"
    severity: high
    
  - name: code_review_coverage
    alert: "(merge_requests_reviewed / merge_requests_total) < 0.90"
    severity: medium
    
  - name: sast_scan_coverage
    alert: "(commits_scanned / commits_total) < 1.0"
    severity: critical
    
  - name: image_scan_coverage
    alert: "(images_scanned / images_pushed) < 1.0"
    severity: critical
```

---

## 19.7 Incident Response Integration

Use audit logs for incident investigation:

```bash
# Incident: Unauthorized access attempt
# Time: 2025-01-15 14:30:00

# Query Teleport session logs
tctl audit get \
  --from=2025-01-15T14:00:00Z \
  --to=2025-01-15T15:00:00Z | \
  jq '.[] | select(.event=="user_login" and .status=="failed")'

# Result:
# {
#   "timestamp": "2025-01-15T14:30:45Z",
#   "event": "user_login",
#   "user": "unknown",
#   "ip": "203.0.113.45",
#   "reason": "Invalid credentials",
#   "status": "failed"
# }

# Query Vault audit log
curl -s -H "X-Vault-Token: $VAULT_TOKEN" \
  https://vault.devsecops.com/v1/sys/audit | \
  grep -A5 "203.0.113.45"

# Query HAProxy logs
grep "203.0.113.45" /var/log/haproxy.log

# Complete incident timeline reconstructed from audit trail
```

---

## 19.8 Chapter Summary

|Chapter|Focus|Deliverable|
|---|---|---|
|**18**|SLSA Framework|Image signing, provenance attestation|
|**19**|Compliance|SBOM, SLA tracking, audit reports|

💡 **Key Takeaway — Complete Traceability & Compliance:**

Every artifact has:

- ✅ Attestation (who built it, when, how)
- ✅ SBOM (what's in it, dependencies)
- ✅ Signature (cryptographically verified)
- ✅ Audit trail (all operations logged)
- ✅ Compliance mapping (SOC2/PCI/HIPAA ready)

---

_DevSecOps Lab Book — Zero to Hero | Chapters 18-19 | Phase 4: Advanced Complete_