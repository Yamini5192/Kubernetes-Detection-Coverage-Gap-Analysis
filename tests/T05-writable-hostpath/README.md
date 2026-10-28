# T05-writable-hostpath

## Objective
Evaluate how effectively Falco's default rules detect pods deployed with sensitive host filesystem mounts (`hostPath`).

> **Lab-only and sensitive:** This simulation mounts the node's `/etc` directory into a temporary pod. Run only in the disposable assessment cluster, do not write to the mount, and remove the pod immediately after the observation window.

## Preconditions
- Target: `falco-lab` (`kind-falco-lab`).
- Telemetry: Active K8s audit webhook receiver (`falco-audit`) and syscall engine (`falco`).

## Procedure
1. Record UTC test start time.
2. Deploy a pod configured with a `hostPath` volume mounting `/etc`.
3. Monitor Falco audit and syscall logs for alerts within the 60-second window.
4. Record observed alerts, rule names, and calculate latency.
5. Delete the test pod immediately.

## Success criteria
- Technique execution: The pod is created and accepted by the Kubernetes API server.
- Detection: An alert identifies the deployment of a pod with a sensitive host volume mount within 60 seconds.
- Cleanup: The pod is completely terminated.

## Evidence
- Evidence log (local archive): `evidence/T05/T05-writable-hostpath-2026-10-22.txt`

## Outcome
- Test status: Completed
- Technique execution: Succeeded (pod `t05-hostpath-test` accepted and scheduled).
- Default alert: `Create Sensitive Mount Pod` (Priority Warning).
- Alert latency: ~1.16 seconds.
- Classification: **Specific** (Detected by default Falco audit rules).
- Retest: Not required (detected by default configuration).
