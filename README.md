# Cert-Manager Controller - CleanStart Container

Cert-manager-controller is an automated certificate management solution for Kubernetes clusters. It simplifies the process of obtaining, renewing and using SSL/TLS certificates from various issuers including Let's Encrypt, HashiCorp Vault, and Venafi. This CleanStart image provides the core cert-manager functionality with enhanced security features, including non-root execution, dropped capabilities, and privilege escalation prevention, making it suitable for security-conscious Kubernetes deployments.

**📌 Base Foundation:** Security-hardened, minimal base OS designed for enterprise containerized environments from CleanStart Registry.

**Image Path:** `ghcr.io/cleanstart-containers/cert-manager-controller`

**Registry:** CleanStart Registry

---

## Overview

The `ghcr.io/cleanstart-containers/cert-manager-controller:latest-dev` image provides a drop-in replacement for the standard cert-manager controller with security hardening applied. The Controller is the main orchestrator that manages the entire certificate lifecycle in Kubernetes - it watches for Certificate and CertificateRequest resources, coordinates with Issuers and ClusterIssuers to obtain certificates, manages certificate renewal, and updates Kubernetes secrets with certificate data. This CleanStart image enables automated TLS certificate provisioning and renewal without manual intervention while maintaining strict security posture.

**Image:** `ghcr.io/cleanstart-containers/cert-manager-controller:latest-dev`  
**Digest:** `sha256:f2b6417563efb81bf56973e773f1c947de085ca1682da1caaf8637be7f4db32f`

**Technical Specifications:**
- **Binary Location:** `/usr/bin/controller`
- **User:** `clnstrt` (UID 1000) - CleanStart non-root user
- **Architecture:** `amd64`
- **OS:** `linux`
- **Image Size:** ~505MB

---

## Key Features

Core capabilities and strengths of this container:

- Automated certificate issuance and renewal
- Certificate lifecycle management (creation, renewal, expiration handling)
- Multiple issuer support (ACME, CA, Vault, Venafi, etc.)
- CertificateRequest resource management and coordination
- Integration with Issuers and ClusterIssuers for certificate issuance
- ACME challenge management for Let's Encrypt and other ACME-compatible CAs
- Automatic certificate renewal before expiration
- Secret management for storing certificate data (tls.crt, tls.key, ca.crt)
- Leader election support for high availability deployments
- Metrics endpoint for monitoring and observability (Prometheus compatible)
- Kubernetes-native certificate management
- Lightweight and efficient operation with minimal resource requirements
- Secure certificate handling and storage

---

## Common Use Cases

Typical scenarios where this container excels:

- Automated TLS certificate provisioning for Kubernetes services
- Let's Encrypt certificate automation for public-facing applications
- Internal CA certificate management for service-to-service communication
- Certificate renewal automation to prevent expiration
- Multi-issuer certificate management (ACME, CA, Vault, etc.)
- Multi-domain SSL certificate management
- Development and staging environment certificate automation
- CI/CD pipeline certificate provisioning
- Microservices architecture with automated certificate management
- Ingress controller TLS certificate automation
- Service mesh certificate provisioning
- Enterprise PKI infrastructure automation

---

## What Cert-Manager Controller Does

The Controller operates as the main orchestrator that manages certificate resources:

1. **Monitors Certificates**: Watches Certificate resources across all namespaces to detect new certificate requests
2. **Creates CertificateRequests**: Generates CertificateRequest resources based on Certificate specifications
3. **Coordinates with Issuers**: Communicates with Issuers and ClusterIssuers to request certificate issuance
4. **Manages ACME Challenges**: For ACME issuers (like Let's Encrypt), manages Order and Challenge resources for domain validation
5. **Processes Certificate Data**: Receives issued certificates and stores them in Kubernetes secrets
6. **Handles Renewal**: Monitors certificate expiration and automatically renews certificates before they expire
7. **Updates Secrets**: Maintains Kubernetes secrets with current certificate data (tls.crt, tls.key, ca.crt)
8. **Leader Election**: Uses leader election to ensure only one instance is active in high availability deployments

---

## CleanStart Security Features

The `ghcr.io/cleanstart-containers/cert-manager-controller:latest-dev` image implements multiple layers of security:

### CleanStart Security Enhancements

1. **Non-Root Execution with Dedicated User**: The image runs as the `clnstrt` user (UID 1000), a dedicated non-root user specifically created for CleanStart containers. This eliminates the risk of root privilege exploitation.

2. **Complete Capability Dropping**: All Linux capabilities are dropped at the container level, ensuring the controller cannot perform privileged operations even if compromised.

3. **Privilege Escalation Prevention**: The image is configured with `allowPrivilegeEscalation: false`, preventing any process from escalating privileges through setuid binaries or other mechanisms.

4. **Pre-Configured SSL/TLS**: The image includes a complete SSL certificate bundle at `/etc/ssl/certs/ca-certificates.crt`, enabling secure communication with the Kubernetes API server and external certificate issuers without requiring additional configuration.

5. **Kubernetes Security Context Integration**: The deployment uses Kubernetes security contexts to enforce non-root execution and prevent privilege escalation at the pod level, providing defense in depth.

### Security Features Summary

- **Non-Root Execution**: Runs as dedicated `clnstrt` user (UID 1000), eliminating root privilege risks
- **Capability Dropping**: All Linux capabilities are dropped, preventing privileged operations
- **Privilege Escalation Prevention**: `allowPrivilegeEscalation: false` prevents privilege escalation attacks
- **Pre-Configured SSL/TLS**: Complete SSL certificate bundle pre-configured for secure communication
- **Kubernetes Security Context**: Pod-level security context enforces non-root execution and prevents privilege escalation
- **Secure Certificate Storage**: Certificates are stored securely in Kubernetes secrets with proper RBAC
- **Least Privilege RBAC**: ClusterRole permissions follow the principle of least privilege
- **Leader Election**: Ensures only one active instance, reducing attack surface
- **Defense in Depth**: Multiple security layers provide comprehensive protection

These security features make the CleanStart image suitable for production environments with strict security requirements, compliance needs, or security-sensitive workloads.

---

## Getting Started

### Pull Latest Image

Download the container image from the registry:
```bash
docker pull ghcr.io/cleanstart-containers/cert-manager-controller:latest
docker pull ghcr.io/cleanstart-containers/cert-manager-controller:latest-dev
```

### Basic Run

Run the container with basic configuration:
```bash
docker run -it --name cert-manager ghcr.io/cleanstart-containers/cert-manager-controller:latest --v=2 --cluster-resource-namespace=cert-manager
```

### Production Deployment

Deploy with production security settings:
```bash
docker run -d --name cert-manager-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  -v /etc/cert-manager:/etc/cert-manager \
  ghcr.io/cleanstart-containers/cert-manager-controller:latest
```

### Volume Mount

Mount local directory for persistent data:
```bash
docker run -v /etc/cert-manager:/etc/cert-manager \
  -v /var/run/cert-manager:/var/run/cert-manager \
  ghcr.io/cleanstart-containers/cert-manager-controller:latest
```

### Port Forwarding

Run with custom port mappings:
```bash
docker run -p 9402:9402 ghcr.io/cleanstart-containers/cert-manager-controller:latest
```

---

## Configuration

### Environment Variables

Configuration options available through environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `SSL_CERT_FILE` | `/etc/ssl/certs/ca-certificates.crt` | SSL certificate path for secure communication with Kubernetes API server and external issuers |
| `PATH` | `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` | Standard PATH environment variable |
| `POD_NAME` | - | Automatically populated from Kubernetes pod metadata via downward API |
| `POD_NAMESPACE` | - | Automatically populated from Kubernetes pod metadata via downward API |
| `NAMESPACE` | `cert-manager` | Namespace for cert-manager deployment |
| `LOG_LEVEL` | `2` | Logging verbosity level |
| `LEADER_ELECTION_NAMESPACE` | `kube-system` | Namespace for leader election |
| `ACME_HTTP01_SOLVER_IMAGE` | `cert-manager-acmesolver` | HTTP01 solver image |

### RBAC Permissions

The Controller requires cluster-scoped permissions to:

- Read and create secrets (to store certificates)
- Read and update configmaps and events
- Manage cert-manager resources (certificates, certificaterequests, issuers, clusterissuers)
- Manage ACME resources (orders, challenges)
- Read CustomResourceDefinitions (to check if cert-manager CRDs are installed)
- Manage leader election leases (for high availability)

---

## Security & Best Practices

### Recommended Security Context

Recommended security context for Kubernetes deployments:
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1001
  fsGroup: 1001
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```

### Best Practices

- Deploy the Controller with proper resource limits to ensure predictable resource usage
- Monitor Controller logs for certificate processing success rates and troubleshooting
- Use the metrics endpoint for Prometheus monitoring and alerting
- Keep the container image updated with the latest security patches
- Implement proper network policies if required by your security policies
- Review Certificate resources to ensure proper issuer configuration
- Enable leader election for high availability deployments
- Monitor certificate expiration and renewal status
- Configure appropriate certificate renewal windows to prevent expiration
- Use ClusterIssuers for cluster-wide certificate management
- Use RBAC policies to restrict certificate management access
- Implement proper secret management for issuer credentials
- Regular audit of certificate requests and issuance
- Monitor certificate expiration and renewal events
- Use secure transport for certificate private keys
- Enable validation webhooks for certificate requests

---

## Observability

### Logging

The Controller provides structured logging that includes:

- Controller startup and initialization events
- Certificate resource monitoring and processing events
- CertificateRequest creation and status updates
- Issuer coordination and certificate issuance events
- ACME challenge management (for ACME issuers)
- Certificate renewal and expiration handling
- Secret update events
- Leader election status and lease acquisition
- Error conditions and failure reasons

### Metrics

The Controller exposes a Prometheus metrics endpoint (port 9402), providing observability into:

- Controller reconciliation rates
- Certificate processing counts and success rates
- CertificateRequest processing statistics
- Certificate renewal events
- Error rates and failure metrics
- Leader election status
- Issuer coordination metrics

### Health Checks

Health check functionality allows Kubernetes to monitor Controller readiness. The metrics endpoint can be used for Kubernetes liveness and readiness probes, enabling proper integration with Kubernetes monitoring systems.

---

## Kubernetes Deployment

The `kubernetes/` directory contains a complete, production-ready Kubernetes deployment:

- `deployment.yaml` - Complete deployment manifest (ServiceAccount, ClusterRole, ClusterRoleBinding, Deployment)
- `README.md` - Comprehensive deployment guide with step-by-step instructions, testing procedures, and troubleshooting

For deployment instructions, configuration details, and troubleshooting guides, see the `kubernetes/README.md` file in the `kubernetes/` directory.

---

## Resources
* **Official Documentation**: https://example.com/docs/cert-manager-controller
* **Provenance / SBOM / Signature**: https://images.cleanstart.com/images/cert-manager-controller
* **Docker Hub**: https://hub.docker.com/r/cleanstart/cert-manager-controller
* **CleanStart All Images**: https://images.cleanstart.com/images/cert-manager-controller/details
* **CleanStart Community Images**: https://hub.docker.com/u/cleanstart


---

## Vulnerability Disclaimer

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

**Security remains a shared responsibility:** CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.
