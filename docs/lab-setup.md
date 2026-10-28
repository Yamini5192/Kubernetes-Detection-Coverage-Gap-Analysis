# Lab Setup

## Reference environment
- Host: Docker Desktop with WSL2
- Kubernetes: a local kind cluster (the assessment used `falco-lab`)
- Falco: 0.44.1, Helm chart 9.1.0
- Runtime driver: modern eBPF
- Supporting tools: kubectl, Helm, Kubescape, kube-bench, Trivy

These details are the recorded starting point and must be revalidated before test execution.

## Safety boundary
Operate only on a disposable local kind cluster created for this assessment. Never target a cloud, shared, or production cluster. Preserve existing workloads. Use fake credentials and tightly scoped temporary resources. Some scenarios intentionally exercise hostPath mounts and static-pod manifests; review their cleanup and node impact before running them.

## Telemetry caveat
Kubernetes API operations require an available API audit telemetry path for audit-based detection. Confirm end-to-end delivery before interpreting tests such as deleting Events or listing Secrets. Do not log Secret contents; metadata-level audit logging is preferred for Secret access tests.
