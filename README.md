# Insecure Demo Apps

Deliberately vulnerable Kubernetes deployments for testing Red Hat Advanced Cluster Security (RHACS) policy violations and CVE detection.

**WARNING: These manifests are intentionally insecure. Do not deploy in production.**

## Deployments

| App | Image | Purpose |
|-----|-------|---------|
| `vuln-nginx` | `nginx:1.14` | Old nginx with hundreds of fixable CVEs, privileged container, secrets in env vars, hostPath mount |
| `vuln-log4j` | `elasticsearch:7.16.1` | Log4Shell (CVE-2021-44228) vulnerable image, runs as root |
| `vuln-shell` | `ubuntu:18.04` | EOL Ubuntu, privileged with SYS_ADMIN/NET_ADMIN caps, hostNetwork, hostPID, docker socket + host root mounts |

## ACS Policies Triggered

- Fixable CVEs (Critical/High/Medium)
- Log4Shell: CVE-2021-44228
- Privileged containers
- Containers running as root (UID 0)
- Environment variables with secrets
- HostPath volume mounts
- Host network / Host PID
- Dangerous capabilities (SYS_ADMIN, NET_ADMIN, SYS_PTRACE)
- Docker socket mount
- No read-only root filesystem
- Writable host mount

## Usage

### OpenShift

On OpenShift, the default SCC will block these pods. Grant the privileged SCC first:

```bash
oc apply -f vulnerable-app.yaml
oc adm policy add-scc-to-user privileged -z default -n insecure-demo
oc rollout restart deployment -n insecure-demo
```

### Kubernetes

```bash
kubectl apply -f vulnerable-app.yaml
```

## Cleanup

```bash
# OpenShift
oc delete namespace insecure-demo

# Kubernetes
kubectl delete namespace insecure-demo
```
