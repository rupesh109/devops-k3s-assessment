
# Infrastructure DevOps Intern Assessment – Synthlane

## Environment
- Cloud Provider: Hetzner Cloud
- OS: Ubuntu 24.04
- Access: SSH (root user)
- Cluster: Single-node k3s

---

## Part 1: VM & Cluster Setup

### Docker Installation
Docker was installed using the official Docker repository. The Docker service was enabled and verified.

**Validation:**
```bash
docker version
docker ps
Kubernetes (k3s) Setup

A single-node k3s cluster was installed. kubectl was configured and verified for the root user.

Validation:
kubectl get nodes
kubectl get pods -A

xplanation – Suitability for Early-Stage Startup:

A single-node k3s setup is suitable for an early-stage startup because it is lightweight, easy to operate, and cost-effective. It provides core Kubernetes primitives while keeping operational complexity low. However, it is not suitable for high availability or fault tolerance and would need to evolve into a multi-node cluster as reliability and scale requirements grow.
Part 2: Application Deployment (Helm)

The Open WebUI application was deployed using Helm v3.

Release Name: webui

Namespace: openwebui

Service Type: ClusterIP

Dry-run performed before installation

Commands:

helm repo add open-webui https://helm.openwebui.com/
helm repo update

kubectl create namespace openwebui

helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --dry-run

helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP

Validation:

kubectl get all -n openwebui

Part 3: Authentication Configuration (OIDC)

OIDC was configured using a placeholder issuer URL as no real identity provider was deployed.

values-oidc.yaml

oidc:
  clientId: "test"
  clientSecret: ""
  issuer: "https://example.com/auth/realms/hyperplane/.well-known/openid-configuration"
  scopes:
    - openid
    - profile
    - email

Applied using:

helm upgrade webui open-webui/open-webui \
  --namespace openwebui \
  --values values-oidc.yaml
Part 4: Debugging (Intentional Failure)
Observed Behavior

The Open WebUI pod reached a Running and Ready state.

The application did not crash at startup.

Startup logs showed normal initialization and database migrations.

Root Cause

The application failed at runtime authentication, not at startup. Open WebUI initializes OIDC lazily and attempts to fetch the OpenID configuration only when authentication is triggered. Since the configured issuer endpoint was invalid/unreachable and no identity provider was deployed, authentication could not succeed.

Diagnosis

Verified pod health using kubectl get pods

Inspected logs using kubectl logs

Confirmed that failure occurred during authentication flows rather than container initialization

Fix

OIDC configuration was reverted/disabled until a valid identity provider is available.

Why This Works

Removing the dependency on an unavailable external authentication service allows the application to operate normally while authentication infrastructure is still under development.

Part 5: Ownership Questions
1. Production Readiness

Top 5 Risks Before Going Live

Single-node cluster (no high availability)

No backups or restore testing

No monitoring or alerting

Secrets stored improperly

No autoscaling or resource limits

First 2 Things to Fix

Add monitoring, logging, and alerting

Implement backups and restore validation

2. Failure Scenario (10x Traffic Spike, Node Down at 2 AM)

What breaks first: CPU/memory exhaustion causing pod evictions

Recovery: Restart node, reduce load, scale services if possible

Next day changes: Add resource limits, autoscaling, and move toward multi-node cluster

3. Security & Secrets

Secrets should be managed using Kubernetes Secrets or an external secret manager

Secrets must never be committed to Git

API keys, tokens, and credentials should be rotated regularly

4. Backups & Recovery

Data to back up: application data, persistent volumes, and configuration

Backup frequency: daily (or more frequently for critical data)

Recovery testing: periodic restore tests in a non-production environment

5. Cost Ownership (Hetzner)

Use small instances and scale gradually

Avoid managed services and over-engineering early

Move away from k3s when high availability, autoscaling, and fault tolerance become mandatory
