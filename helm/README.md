# Helm Configurations

This directory contains the Helm values files used to deploy and configure Falco within the local `falco-lab` Kind cluster.

> **Lab-only configuration:** `falco-audit-values.yaml` exposes the audit receiver as a NodePort using plain HTTP, without receiver authentication. Restrict network access to the lab control plane and receiver. Do not deploy this receiver configuration to a shared or production cluster without adding appropriate network controls, authentication, and transport protection.

## Files

- **`falco-release-values.yaml`**: Helm values for the runtime Falco DaemonSet (modern eBPF). Mounts the syscall custom rule `t04-static-pod.yaml`.
- **`falco-audit-values.yaml`**: Helm values for the Falco Kubernetes audit receiver. Enables the `k8saudit` plugin (`k8saudit-rules:0.16`), exposes NodePort `30007` (port `9765`), and mounts k8saudit custom rules for T01, T03, and T07.

## Deployment Commands

### Upgrading the Runtime DaemonSet with Custom Rules
```bash
helm upgrade falco falcosecurity/falco \
  --namespace falco \
  --version 9.1.0 \
  -f helm/falco-release-values.yaml
```

### Deploying / Validating the K8s Audit Receiver
```bash
# Render templates to verify configuration:
helm template falco-audit falcosecurity/falco \
  --version 9.1.0 \
  --namespace falco \
  --values helm/falco-audit-values.yaml

# Deploy the audit receiver:
helm upgrade --install falco-audit falcosecurity/falco \
  --version 9.1.0 \
  --namespace falco \
  --values helm/falco-audit-values.yaml
```
