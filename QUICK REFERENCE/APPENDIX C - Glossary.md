# APPENDIX C — DevSecOps Glossary & Terminology
## DevSecOps Lab Book — Essential Terms & Definitions

---

## Core Concepts

**DevSecOps**: Integration of security into DevOps practices, shifting security left in the development lifecycle

**Defense in Depth**: Multiple security layers to protect systems; if one fails, others remain

**Least Privilege**: Users/processes get minimum permissions necessary to perform their function

**Zero Trust**: Security model assuming no trust by default; all access requires authentication & authorization

## Infrastructure Terms

**IaC (Infrastructure as Code)**: Define infrastructure using code (Terraform, Ansible)

**Idempotent**: Operation produces same result regardless of how many times executed

**Immutable Infrastructure**: Infrastructure treated as unchangeable; replaced rather than updated

**Golden Image**: Pre-configured VM/container image used as base for deployments

## Kubernetes Terms

**Control Plane**: Kubernetes brain (API server, scheduler, etcd, controller manager)

**kubelet**: Agent on each node managing pod lifecycle

**Pod**: Smallest Kubernetes unit; contains one or more containers

**Service**: Stable endpoint for pod access; enables load balancing

**Ingress**: External HTTP(S) routing to services

**Helm**: Package manager for Kubernetes; templates & charts

**RBAC**: Role-Based Access Control; define who can do what

## Security Terms

**mTLS (Mutual TLS)**: Both client & server authenticate each other

**RBAC**: Role-Based Access Control; users granted roles, roles have permissions

**ABAC**: Attribute-Based Access Control; decisions based on attributes

**SAST**: Static Application Security Testing; code analysis without execution

**DAST**: Dynamic Application Security Testing; testing running application

**Vulnerability**: Software flaw allowing unintended access/behavior

**Exploit**: Code/technique that uses vulnerability to compromise system

**Threat**: Potential negative impact; threat = vulnerability + attacker

## Compliance Terms

**SOC 2**: Service Organization Control framework (security, availability, confidentiality)

**PCI-DSS**: Payment Card Industry Data Security Standard

**HIPAA**: Health Insurance Portability & Accountability Act

**GDPR**: General Data Protection Regulation (EU privacy law)

**ISO 27001**: International information security standard

**CIS Benchmarks**: Center for Internet Security security best practices

**SLSA**: Supply chain Levels for Software Artifacts; software integrity framework

## Operations Terms

**RTO**: Recovery Time Objective; maximum acceptable downtime

**RPO**: Recovery Point Objective; maximum acceptable data loss

**MTTR**: Mean Time To Repair; average time to fix issues

**Incident**: Security event requiring response

**Alert**: Notification of suspicious activity

**Baseline**: Normal system behavior reference point

**Anomaly**: Deviation from baseline

## Data Terms

**Encryption in Transit**: Data encrypted while moving (TLS)

**Encryption at Rest**: Data encrypted while stored

**Key Rotation**: Regularly changing cryptographic keys

**Tokenization**: Replace sensitive data with non-sensitive token

**PII**: Personally Identifiable Information (private data)

**ePHI**: Electronic Protected Health Information

**Data Classification**: Categorizing data by sensitivity

## Deployment Terms

**Blue-Green**: Two identical production environments; switch traffic between them

**Canary**: Deploy to small percentage of users first; monitor before full rollout

**Rolling Update**: Gradually replace old instances with new ones

**Feature Flag**: Toggle features on/off without code changes

**Trunk-Based Development**: All developers commit to main branch; feature flags manage incomplete work

## Monitoring Terms

**Metric**: Quantifiable measurement (CPU, memory, requests/sec)

**Log**: Record of events in system

**Trace**: Path of request through distributed system

**Alert**: Notification when metric exceeds threshold

**Baseline**: Normal values for comparison

**Anomaly Detection**: Identifying unusual patterns

**Dashboard**: Visual display of metrics/data

---

*DevSecOps Lab Book — Appendix C: Glossary*

