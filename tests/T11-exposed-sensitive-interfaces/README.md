# T11 — Exposed Sensitive Interfaces

## Objective

Assess whether Falco alerts when a temporary workload uses host networking to reach the kind node's kubelet HTTPS endpoint. The test is limited to the existing lab node and does not change kubelet configuration or expose a new service.

## Safety and preconditions

- Use only the existing `falco-lab` kind cluster and `falco-research` namespace.
- Confirm the audit receiver and runtime Falco pod are healthy.
- Use the cached `curlimages/curl:latest` image, no privileged context, and a single request to `https://127.0.0.1:10250/pods`.
- Do not query port 10255 or publish any returned pod data. Discard the response body and record only the HTTP status.
- Stop if host-network pod creation is disallowed or the node endpoint is not listening.

## Procedure

1. Record the test start time and current Falco pod identities.
2. Create a temporary, non-privileged pod in `falco-research` with `hostNetwork: true`, `restartPolicy: Never`, and the cached curl image. Run `sleep 3600` as its command.
3. Wait for Ready and record the successful `Create HostNetwork Pod` audit alert, if any.
4. Record the request timestamp and issue exactly one request from the pod:

   ```powershell
   kubectl exec t11-interface-probe -n falco-research -- curl -sk --max-time 3 -o /dev/null -w "%{http_code}" https://127.0.0.1:10250/pods
   ```

5. Capture audit and syscall alerts for 60 seconds from the pod creation/request sequence. Save only rule names, priorities, timestamps, and the HTTP status; do not save the response body.
6. Delete the pod and verify it is absent.

## Classification

- Technique succeeds if the request reaches the local endpoint and returns an HTTP status (including 401/403). A connection failure makes the interface-access portion inconclusive.
- A `Create HostNetwork Pod` alert is a generic signal for the access setup; only an alert that identifies the kubelet endpoint/request is specific to this technique.
- No alert can be called a miss only if the action succeeded and the relevant telemetry was available.

## Status

Completed (28 October 2026). The kubelet endpoint returned HTTP 401, confirming reachability; the response body was discarded. Falco emitted `Pod started using host network` (Warning) approximately 0.23 seconds after test start. This is a generic alert for the setup, not a specific alert on the kubelet request. Cleanup was confirmed.

Evidence: `evidence/T11/T11-exposed-sensitive-interfaces-2026-10-28.txt` (local archive).
