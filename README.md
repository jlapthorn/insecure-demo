# Insecure Demo Apps

Deliberately vulnerable Kubernetes deployments for testing Red Hat Advanced Cluster Security (RHACS) policy violations and CVE detection.

**WARNING: These manifests are intentionally insecure. Do not deploy in production.**

## Files

| File | Contents |
|------|----------|
| `vulnerable-app.yaml` | Core apps (nginx, elasticsearch/Log4j, ubuntu shell) |
| `vulnerable-extras.yaml` | Extended apps (databases, web frameworks, RBAC abuse, crypto miner) |

## Deployments

### Core (`vulnerable-app.yaml`)

| App | Image | Key Violations |
|-----|-------|----------------|
| `vuln-nginx` | `nginx:1.14` | Hundreds of fixable CVEs, privileged, secrets in env vars, hostPath mount |
| `vuln-log4j` | `elasticsearch:7.16.1` | Log4Shell (CVE-2021-44228), runs as root |
| `vuln-shell` | `ubuntu:18.04` | Privileged, SYS_ADMIN/NET_ADMIN caps, hostNetwork, hostPID, docker socket + host root mounts |

### Extended (`vulnerable-extras.yaml`)

| App | Image | Key Violations |
|-----|-------|----------------|
| `vuln-spring4shell` | `spring-core-rce-2022-22965` | Spring4Shell (CVE-2022-22965), runs as root |
| `vuln-redis` | `redis:5.0.0` | Old Redis with no authentication, many CVEs |
| `vuln-postgres` | `postgres:9.6` | EOL PostgreSQL, default credentials in env vars |
| `vuln-node` | `node:10-slim` | EOL Node.js 10, prototype pollution CVEs, runs as root |
| `vuln-httpd` | `httpd:2.4.29` | Old Apache httpd, many CVEs including path traversal |
| `vuln-mongodb` | `mongo:4.0` | Old MongoDB with no auth, runs as root |
| `vuln-python` | `python:2.7-slim` | EOL Python 2.7, AWS secret key in env vars |
| `vuln-mysql` | `mysql:5.6` | EOL MySQL, weak root password in env vars |
| `vuln-miner` | `ubuntu:18.04` | Crypto miner lookalike, installs packages at runtime, stratum port |
| `vuln-kubectl` | `bitnami/kubectl:1.23` | cluster-admin ServiceAccount, RBAC over-privilege |

## ACS Policies Triggered

**Image Vulnerabilities:**
- Fixable CVEs (Critical/High/Medium/Low)
- Log4Shell: CVE-2021-44228
- Spring4Shell: CVE-2022-22965
- EOL operating systems (Ubuntu 18.04, Python 2.7)

**Container Configuration:**
- Privileged containers
- Containers running as root (UID 0)
- AllowPrivilegeEscalation enabled
- Dangerous capabilities (SYS_ADMIN, NET_ADMIN, SYS_PTRACE)
- No read-only root filesystem
- Latest tag used

**Secrets & Credentials:**
- Environment variables containing secrets (DATABASE_PASSWORD, API_KEY, AWS_SECRET_ACCESS_KEY)
- Default/weak database passwords

**Host Access:**
- HostPath volume mounts (/etc, /)
- Docker socket mount
- Host network enabled
- Host PID namespace

**RBAC:**
- ServiceAccount with cluster-admin ClusterRoleBinding
- Automounted service account tokens

**Runtime:**
- Crypto mining indicators (stratum port, mining pool env vars)
- Package manager execution in container (apt-get)

## Usage

### Deploy everything on OpenShift

```bash
oc apply -f vulnerable-app.yaml
oc apply -f vulnerable-extras.yaml
oc adm policy add-scc-to-user privileged -z default -n insecure-demo
oc adm policy add-scc-to-user privileged -z overprivileged-sa -n insecure-demo
oc rollout restart deployment -n insecure-demo
```

### Deploy everything on Kubernetes

```bash
kubectl apply -f vulnerable-app.yaml
kubectl apply -f vulnerable-extras.yaml
```

### Deploy selectively

Each file uses the same `insecure-demo` namespace. You can apply either file independently.

## Cleanup

```bash
# OpenShift
oc delete namespace insecure-demo
oc delete clusterrolebinding insecure-demo-cluster-admin

# Kubernetes
kubectl delete namespace insecure-demo
kubectl delete clusterrolebinding insecure-demo-cluster-admin
```
