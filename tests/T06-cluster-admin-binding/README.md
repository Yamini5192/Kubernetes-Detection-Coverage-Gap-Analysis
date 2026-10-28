# T06-cluster-admin-binding

## Objective
Evaluate how effectively Falco's default rules detect privilege escalation through the assignment of `cluster-admin` permissions via a ClusterRoleBinding.

> **Lab-only and privilege-changing:** This simulation grants cluster-admin to a test identity. Run only in the disposable assessment cluster and delete the binding immediately after the observation window.

## Preconditions
- Target: `falco-lab` (`kind-falco-lab`).
- Telemetry: Active K8s audit webhook receiver (`falco-audit`).

## Procedure
1. Record UTC test start time.
2. Create a test `ClusterRoleBinding` linking a test service account to the built-in `cluster-admin` role.
3. Monitor Falco audit logs for alerts within the 60-second window.
4. Record observed alerts, rule names, and calculate latency.
5. Delete the test `ClusterRoleBinding` immediately.

## Success criteria
- Technique execution: The ClusterRoleBinding is created and persisted by the API server.
- Detection: Falco triggers an alert explicitly identifying the binding to `cluster-admin`.
- Cleanup: The ClusterRoleBinding is removed completely.

## Evidence
- Evidence log (local archive): `evidence/T06/T06-cluster-admin-binding-2026-10-22.txt`

## Outcome
- Test status: Completed
- Technique execution: Succeeded (`clusterrolebinding.rbac.authorization.k8s.io/t06-test-binding` created).
- Default alert: `Attach to cluster-admin Role` (Priority Warning).
- Alert latency: ~0.57 seconds.
- Classification: **Specific** (Detected by default Falco audit rules).
- Retest: Not required (detected by default configuration).
