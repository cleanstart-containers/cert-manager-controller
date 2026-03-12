# Cert-Manager Controller - Kubernetes Deployment Guide

Complete Kubernetes deployment guide for the CleanStart Cert-Manager Controller container. The Controller manages the lifecycle of certificates, certificate requests, and coordinates with issuers to obtain certificates.

## Image Details

**Image:** `cleanstart/cert-manager-controller:latest-dev`  
**Digest:** `sha256:f2b6417563efb81bf56973e773f1c947de085ca1682da1caaf8637be7f4db32f`

**Key Features:**
- Binary: `/usr/bin/controller` (entrypoint from docker inspect)
- User: Non-root (UID 1000, user: `clnstrt`)
- Architecture: `amd64`, OS: `linux`
- SSL Certificates: `/etc/ssl/certs/ca-certificates.crt`

## Prerequisites

1. Kubernetes cluster (Kind, minikube, k3s, GKE, EKS, AKS, etc.)
2. `kubectl` installed and configured
3. cert-manager CRDs installed:
   ```bash
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.crds.yaml
   ```

## Deployment Steps

### Step 1: Verify Prerequisites

```bash
kubectl cluster-info
kubectl get nodes
kubectl get crd | grep cert-manager
```

**Expected CRDs:** `certificates.cert-manager.io`, `certificaterequests.cert-manager.io`, `issuers.cert-manager.io`, `clusterissuers.cert-manager.io`, `orders.acme.cert-manager.io`, `challenges.acme.cert-manager.io`

### Step 2: Deploy

```bash
kubectl apply -f deployment.yaml
kubectl get pods -l app=cert-manager-controller -w
```

**Expected Output:**
```
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-controller-xxxxxxxxxx-xxxxx  1/1     Running   0          30s
```

## Testing

### Test 1: Pod Health

```bash
kubectl get pods -l app=cert-manager-controller
kubectl logs -l app=cert-manager-controller
```

**Success:** Pod `Running`, logs show no errors and controller watching resources

### Test 2: RBAC Permissions

```bash
kubectl describe clusterrole cert-manager-controller
kubectl auth can-i get certificates --as=system:serviceaccount:default:cert-manager-controller
kubectl auth can-i get certificaterequests --as=system:serviceaccount:default:cert-manager-controller
kubectl auth can-i create secrets --as=system:serviceaccount:default:cert-manager-controller
```

**Expected:** All commands return `yes`

### Test 3: Image Configuration

```bash
kubectl get pod -l app=cert-manager-controller -o jsonpath='{.items[0].spec.containers[0].image}'
kubectl exec -it deployment/cert-manager-controller -- ls -la /usr/bin/controller
kubectl exec -it deployment/cert-manager-controller -- env | grep -E "SSL_CERT_FILE|PATH"
kubectl exec -it deployment/cert-manager-controller -- ps aux | grep controller
```

**Expected:** Image `cleanstart/cert-manager-controller:latest-dev`, binary exists at `/usr/bin/controller`, `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt`

### Test 4: Security Context

```bash
kubectl exec -it deployment/cert-manager-controller -- id
kubectl exec -it deployment/cert-manager-controller -- ls -la /etc/ssl/certs/ca-certificates.crt
```

**Expected:** UID 1000 (non-root), `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `runAsNonRoot: true`

### Test 5: Certificate Processing

```bash
# Create self-signed ClusterIssuer
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
EOF

# Create test Certificate
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: test-cert
  namespace: default
spec:
  secretName: test-cert-tls
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
  commonName: example.com
  dnsNames:
  - example.com
EOF

# Verify
kubectl get certificate test-cert
kubectl get secret test-cert-tls
kubectl get certificaterequest
kubectl logs -l app=cert-manager-controller --tail=50
```

**Success:** Certificate `Ready: True`, secret created with `tls.crt` and `tls.key`, CertificateRequest processed

### Test 6: Resource Usage & Image Digest

```bash
kubectl top pod -l app=cert-manager-controller
kubectl get pod -l app=cert-manager-controller -o jsonpath='{.items[0].status.containerStatuses[0].imageID}'
```

## Configuration

### Environment Variables
- `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt`
- `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`
- `POD_NAME`, `POD_NAMESPACE` (auto-set from pod metadata)

### Resource Limits
- **Requests:** CPU: 100m, Memory: 128Mi
- **Limits:** CPU: 500m, Memory: 512Mi

### Security Context
- Non-root user (UID 1000, user: `clnstrt`)
- All capabilities dropped
- No privilege escalation
- Read-only root filesystem: false

## Deployment Components

The `deployment.yaml` includes:

1. **ServiceAccount** - Identity for the pod
2. **ClusterRole & ClusterRoleBinding** - Cluster-wide RBAC permissions for:
   - Secrets, configmaps, events
   - Cert-manager resources (certificates, certificaterequests, issuers, clusterissuers)
   - ACME resources (orders, challenges)
   - CustomResourceDefinitions
   - Leader election leases
3. **Deployment** - Controller binary at `/usr/bin/controller` with security best practices

## Integration

This custom controller image can be integrated with cert-manager in two ways:

### Option 1: Replace Default Controller

Replace the default cert-manager controller with this custom image. This is useful when you need to use a custom-built controller with specific features or modifications.

**Steps:**

1. **Scale down the default controller:**
   ```bash
   kubectl scale deployment cert-manager -n cert-manager --replicas=0
   ```

2. **Update the deployment.yaml namespace:**
   - Change the namespace from `default` to `cert-manager` in the ServiceAccount and ClusterRoleBinding
   - Ensure the ServiceAccount name matches what cert-manager expects (typically `cert-manager`)

3. **Deploy the custom controller:**
   ```bash
   kubectl apply -f deployment.yaml
   ```

4. **Verify the controller is running:**
   ```bash
   kubectl get pods -n cert-manager -l app=cert-manager-controller
   kubectl logs -n cert-manager -l app=cert-manager-controller
   ```

**Important Considerations:**
- Ensure cert-manager CRDs are already installed
- The controller must have the same RBAC permissions as the default controller
- Verify that certificate processing continues to work after replacement
- Monitor logs for any compatibility issues

### Option 2: Test Alongside Default Controller

For testing purposes only, you can run this controller alongside the default one. However, this is **not recommended for production** due to potential conflicts:

**Potential Issues:**
- **Duplicate Processing**: Both controllers may attempt to process the same Certificate resources, causing race conditions
- **Resource Conflicts**: Multiple controllers creating CertificateRequests for the same Certificate
- **Leader Election Conflicts**: If both controllers participate in leader election, they may interfere with each other
- **Unpredictable Behavior**: Certificates may be processed inconsistently or fail unexpectedly

**If testing alongside:**
- Use a different namespace or distinct labels to avoid conflicts
- Monitor both controllers' logs closely
- Be prepared to scale down one controller if issues arise
- Only use this approach in isolated test environments

## Cleanup

```bash
kubectl delete -f deployment.yaml
```

Or individually:
```bash
kubectl delete deployment,serviceaccount cert-manager-controller
kubectl delete clusterrole,clusterrolebinding cert-manager-controller
```

```bash
kubectl delete clusterissuer.cert-manager.io/selfsigned-issuer
kubectl delete certificate.cert-manager.io/test-cert
```

**Note:** ClusterRole and ClusterRoleBinding are cluster-wide - ensure no dependencies before deletion.
