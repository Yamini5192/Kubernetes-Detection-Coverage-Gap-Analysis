# Lab Setup

## Reference environment
- Host: Docker Desktop with WSL2
- Kubernetes: a local kind cluster (the assessment used `falco-lab`)
- Falco: 0.44.1, Helm chart 9.1.0
- Runtime driver: modern eBPF
- Supporting tools: kubectl, Helm, Kubescape, kube-bench, Trivy

These details are the recorded starting point and must be revalidated before test execution.

## Workloads used by network tests
T08 and T12 rely on services from the separately installed Kubernetes Goat lab and the `health-check-service` workload. Those application manifests are not included in this repository. Before running either test, confirm the named services in the procedure exist in the disposable cluster; do not point the tests at unrelated or production services. T08 also requires the controlled proxy manifest in `tests/T08-connect-from-proxy/`.

The procedures use fixed version tags for their small test images. These are version-tagged references, not immutable image digests. For a supply-chain audit or exact rerun, record the resolved image ID/digest from the kind node along with the test run.

## Safety boundary
Operate only on a disposable local kind cluster created for this assessment. Never target a cloud, shared, or production cluster. Preserve existing workloads. Use fake credentials and tightly scoped temporary resources. Some scenarios intentionally exercise hostPath mounts and static-pod manifests; review their cleanup and node impact before running them.

## Telemetry caveat
Kubernetes API operations require an available API audit telemetry path for audit-based detection. Confirm end-to-end delivery before interpreting tests such as deleting Events or listing Secrets. Do not log Secret contents; metadata-level audit logging is preferred for Secret access tests.
