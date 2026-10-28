# T12 — Network Mapping

## Objective

Evaluate Falco's default runtime coverage for a network-discovery tool executed inside a container. The bounded lab procedure uses cached BusyBox `nc` to probe three explicitly named in-cluster Services and ports; it does not scan a subnet or external hosts.

## Preconditions

- Target only the existing `falco-lab` kind cluster.
- Confirm runtime Falco and its modern eBPF driver are healthy.
- Confirm `busybox:1.36` is cached on the kind node and all target Services are available.
- Use one temporary unprivileged pod in `falco-research`.

## Procedure

1. Record the test start time and verify the named Services are present.
2. Create a temporary pod using cached `busybox:1.36`, with `restartPolicy: Never` and command `sleep 3600`.
3. Wait for Ready and record the action timestamp.
4. From the pod, run TCP probes only against these approved service/port pairs:

   ```powershell
   kubectl exec t12-netmap-test -n falco-research -- sh -c 'nc -zvw2 health-check-service.default.svc.cluster.local 80 && nc -zvw2 kubernetes-goat-home-service.default.svc.cluster.local 80 && nc -zvw2 internal-proxy-info-app-service.default.svc.cluster.local 5000'
   ```

5. Capture syscall and audit alerts for 60 seconds from the `nc` execution. Record the command result, alert name, priority, and timestamp. Do not expand the target list or probe external destinations.
6. Delete the pod and verify it is absent.

## Classification

- Technique succeeds if `nc` executes and returns a connection result for the selected service/port.
- A tool-execution alert is generic unless it identifies network mapping; a rule specifically identifying the probe is specific.
- A no-alert outcome is valid only when the action succeeded and syscall telemetry was available.

## Status

Completed (28 October 2026). The original TCP/53 probe succeeded. A bounded retest also connected successfully to the three named in-cluster service/port pairs: health-check (80), Kubernetes Goat home (80), and internal-proxy info-app (5000). Falco emitted a generic `Attach/Exec to pod` Notice at approximately 0.19 seconds; no syscall alert identifying the network probes was observed during the 60-second window. The temporary pod was removed.

Classification: **Generic** for the exec-session signal; the test did not produce a network-mapping-specific detection.
Evidence: `evidence/T12/T12-network-mapping-retest-2026-10-28.txt` and local log captures (local archive).
