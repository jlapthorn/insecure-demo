# Insecure Demo Apps

Deliberately vulnerable Kubernetes deployments for testing Red Hat Advanced Cluster Security (RHACS) policy violations and CVE detection.

**WARNING: These manifests are intentionally insecure. Do not deploy in production.**

## Files

| File | Contents |
|------|----------|
| `vulnerable-app.yaml` | Core apps (nginx, elasticsearch/Log4j, ubuntu shell) |
| `vulnerable-extras.yaml` | Extended apps (databases, web frameworks, RBAC abuse, crypto miner) |

## App Inventory

### Core Apps (`vulnerable-app.yaml`)

| # | App | Image | Notable CVEs | Security Misconfigurations | ACS Policies Triggered |
|---|-----|-------|-------------|---------------------------|----------------------|
| 1 | `vuln-nginx` | `nginx:1.14` (2018) | CVE-2019-9511, CVE-2019-9513, CVE-2021-23017, CVE-2022-41741 + hundreds more | Privileged container, run as root, secrets in env vars (`DATABASE_PASSWORD`, `API_KEY`), hostPath `/etc` mount | Fixable CVEs, Privileged Container, Run as Root, Environment Variable Contains Secret, HostPath Volume Mount |
| 2 | `vuln-log4j` | `elasticsearch:7.16.1` | **CVE-2021-44228 (Log4Shell)**, CVE-2021-45046, CVE-2021-45105 | Run as root, writable root filesystem | Log4Shell, Fixable CVEs, Run as Root |
| 3 | `vuln-shell` | `ubuntu:18.04` (EOL) | CVE-2022-29155, CVE-2021-3711, CVE-2022-2068 + many EOL-related CVEs | Privileged, SYS_ADMIN + NET_ADMIN + SYS_PTRACE caps, hostNetwork, hostPID, docker socket mount, host root `/` mount | Privileged Container, Dangerous Capabilities, Host Network, Host PID, Docker Socket Mount, Mount Sensitive Host Directory |

### Extended Apps (`vulnerable-extras.yaml`)

| # | App | Image | Notable CVEs | Security Misconfigurations | ACS Policies Triggered |
|---|-----|-------|-------------|---------------------------|----------------------|
| 4 | `vuln-spring4shell` | `tomcat:8.5.0` (2016) | **CVE-2020-1938 (Ghostcat)**, CVE-2017-12615, CVE-2017-12617, CVE-2019-0232 + many more | Run as root, writable root filesystem, allowPrivilegeEscalation | Fixable CVEs, Run as Root, AllowPrivilegeEscalation |
| 5 | `vuln-redis` | `redis:5.0.0` (2018) | CVE-2021-32761, CVE-2022-24735, CVE-2022-24736 | No Redis AUTH configured, run as root, writable root filesystem | Fixable CVEs, Run as Root, No Authentication |
| 6 | `vuln-postgres` | `postgres:9.6` (EOL) | CVE-2019-10164, CVE-2020-25695, CVE-2021-23214, CVE-2022-1552 | Default credentials in env vars (`POSTGRES_PASSWORD=postgres123`), run as root | Fixable CVEs, EOL Software, Environment Variable Contains Secret, Run as Root |
| 7 | `vuln-node` | `node:10-slim` (EOL) | CVE-2019-15604, CVE-2020-8174, CVE-2021-22884, prototype pollution CVEs | Run as root, EOL Node.js 10, allowPrivilegeEscalation | Fixable CVEs, EOL Software, Run as Root, AllowPrivilegeEscalation |
| 8 | `vuln-httpd` | `httpd:2.4.29` (2017) | CVE-2021-44790, CVE-2021-26691, CVE-2019-0211, CVE-2021-40438 | Run as root, writable root filesystem | Fixable CVEs, Run as Root |
| 9 | `vuln-mongodb` | `mongo:4.0` (EOL) | CVE-2020-7921, CVE-2021-20330, CVE-2021-32040 | No authentication, run as root, writable root filesystem | Fixable CVEs, EOL Software, Run as Root, No Authentication |
| 10 | `vuln-python` | `python:2.7-slim` (EOL) | CVE-2019-9636, CVE-2019-16056, CVE-2021-3177, CVE-2022-0391 | EOL Python 2.7, AWS secret key in env vars (`AWS_SECRET_ACCESS_KEY`), run as root | Fixable CVEs, EOL Software, Environment Variable Contains Secret, Run as Root |
| 11 | `vuln-mysql` | `mysql:5.6` (EOL) | CVE-2019-2627, CVE-2020-2922, CVE-2021-2007, CVE-2022-21589 | Weak root password in env vars (`root`), run as root | Fixable CVEs, EOL Software, Environment Variable Contains Secret, Run as Root |
| 12 | `vuln-miner` | `ubuntu:18.04` (EOL) | Same EOL Ubuntu CVEs as #3 | Installs packages at runtime (apt-get), stratum mining port 3333, mining pool env vars, run as root | Cryptocurrency Mining, Package Manager Execution, Fixable CVEs, Run as Root |
| 13 | `vuln-kubectl` | `alpine:3.14` (EOL) | CVE-2021-36159, CVE-2022-28391, CVE-2023-5363 + musl/openssl CVEs | `cluster-admin` ClusterRoleBinding, automounted SA token, run as root | RBAC Over-Privilege, Cluster Admin Binding, Fixable CVEs, Run as Root |

## ACS Policy Categories Covered

| Category | Policies | Triggered By |
|----------|----------|--------------|
| **Image Vulnerabilities** | Fixable CVEs (Critical/High/Medium/Low) | All 13 apps |
| **Critical CVEs** | Log4Shell (CVE-2021-44228) | `vuln-log4j` |
| | Ghostcat (CVE-2020-1938) | `vuln-spring4shell` |
| **EOL Software** | Image uses EOL operating system or runtime | `vuln-shell`, `vuln-postgres`, `vuln-node`, `vuln-mongodb`, `vuln-python`, `vuln-mysql`, `vuln-miner` |
| **Container Config** | Privileged Container | `vuln-nginx`, `vuln-shell` |
| | Run as Root (UID 0) | All 13 apps |
| | AllowPrivilegeEscalation | `vuln-nginx`, `vuln-spring4shell`, `vuln-node`, `vuln-miner` |
| | No Read-Only Root Filesystem | All 13 apps |
| | Latest Tag | `vuln-spring4shell` |
| **Capabilities** | SYS_ADMIN, NET_ADMIN, SYS_PTRACE | `vuln-shell` |
| **Secrets** | Secrets in Environment Variables | `vuln-nginx`, `vuln-postgres`, `vuln-python`, `vuln-mysql` |
| | AWS Credentials in Environment | `vuln-nginx`, `vuln-python` |
| **Host Access** | HostPath Volume Mount | `vuln-nginx`, `vuln-shell` |
| | Docker Socket Mount | `vuln-shell` |
| | Host Network | `vuln-shell` |
| | Host PID Namespace | `vuln-shell` |
| **RBAC** | Cluster Admin Role Binding | `vuln-kubectl` |
| | Automounted Service Account Token | `vuln-kubectl` |
| **Runtime** | Cryptocurrency Mining Indicators | `vuln-miner` |
| | Package Manager Execution | `vuln-miner` |

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
