# T01-sidecar-injection

## Objective
Evaluate how effectively Falco's default runtime and Kubernetes API audit rules detect sidecar injection into an active deployment workload.

## Preconditions
- Confirm target cluster is `falco-lab` (`kind-falco-lab`).
- Confirm Falco syscall engine (`falco`) and K8s audit receiver (`falco-audit`) are healthy.
- Target workload: `deployment/sidecar-target` in namespace `falco-research`.
- Use the pinned harmless image `busybox:1.38.0` (pull it into the kind node before the test if it is not cached).
- Record test start timestamp before applying the patch.

## Procedure
1. Verify pre-test state of `deployment/sidecar-target` (1 container: `nginx`).
2. Record UTC test start time.
3. Apply a strategic merge patch to inject a secondary container `injected-sidecar` running `sleep 3600`.
4. Monitor rollout until the patched deployment completes.
5. Verify that the newly spawned pod contains both `nginx` and `injected-sidecar` and reaches Running.
6. Inspect Falco syscall and audit logs across the 60-second detection window.
7. Revert the deployment by removing the injected container definition.
8. Verify rollout of the single-container deployment and cleanup of the test pod.

## Success criteria
- Technique execution: The sidecar container successfully attaches to the pod and reaches Running state.
- Detection: A specific or generic Falco alert identifies the sidecar injection or workload modification within 60 seconds.
- Cleanup: The deployment is reverted to its original single-container spec, and the multi-container pod is terminated.

## Evidence
- Evidence log (local archive): `evidence/T01/T01-sidecar-injection-2026-10-22.txt`
- Custom-rule retest: `evidence/T01/T01-custom-rule-retest-2026-10-26.txt`

## Outcome
- Test status: Completed
- Technique execution: Succeeded (the injected sidecar reached Running in the temporary test pod).
- Telemetry availability: Available (Modern eBPF syscalls and K8s API audit webhook verified active).
- Default alert: None observed in captured logs.
- Classification: **No Detection** (Substantiated detection gap).
- Custom-rule retest: **Detected**. The T01 audit rule emitted Warning alerts during a successful workload patch retest.
- Cleanup: Confirmed. Reverted to single-container `nginx` deployment.
