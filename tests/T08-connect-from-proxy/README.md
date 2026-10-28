# T08 — Connect through a proxy

## Objective

Measure Falco alerts for an HTTP request routed through a forward proxy from a pod to one controlled in-cluster destination. This procedure tests HTTP only; it does not test HTTPS CONNECT or public egress.

## Safety and prerequisites

- Use only the isolated disposable lab and verify the active Kubernetes context.
- Confirm Falco and the modern eBPF driver are healthy.
- `proxy-lab.yaml` deploys a temporary ClusterIP proxy whose configuration allows only `health-check-service.default.svc.cluster.local`; it has no public listener or general-purpose destination access.
- Use the cached `nginx:latest` and `curlimages/curl:latest` images. Do not change the allowlist to a public or production endpoint.

## Procedure

1. Record the UTC start time and current Falco pod readiness.
2. Apply the restricted in-cluster proxy:
   ```bash
   kubectl apply -f tests/T08-connect-from-proxy/proxy-lab.yaml
   kubectl -n falco-research wait --for=condition=Ready pod/t08-forward-proxy --timeout=90s
   ```
3. Start a temporary curl client:
   ```bash
   kubectl -n falco-research run t08-proxy-test --image=curlimages/curl:latest --image-pull-policy=IfNotPresent --restart=Never --command -- sleep 600
   kubectl -n falco-research wait --for=condition=Ready pod/t08-proxy-test --timeout=60s
   ```
4. Record a test timestamp and make one body-discarding request through the proxy:
   ```bash
   kubectl -n falco-research exec t08-proxy-test -- curl --max-time 10 --proxy http://t08-forward-proxy.falco-research.svc.cluster.local:3128 --output /dev/null --silent --show-error --write-out 'http_code=%{http_code} bytes=%{size_download}\n' http://health-check-service.default.svc.cluster.local/
   ```
5. Observe Falco syscall and audit logs for 60 seconds. Distinguish an audit alert for `kubectl exec` from an alert identifying the outbound network connection.
6. Record the proxy access status and matching alerts, then clean up:
   ```bash
   kubectl -n falco-research delete pod t08-proxy-test --wait=true
   kubectl -n falco-research delete -f tests/T08-connect-from-proxy/proxy-lab.yaml --wait=true
   ```
7. Verify the client pod, proxy pod, Service, and ConfigMap are absent.

## Success criteria

- Technique execution succeeds when curl exits successfully with HTTP 200 and the proxy access log confirms the request.
- A generic exec-session audit alert does not count as proxy-specific network detection. Record it as Generic only when applying the project taxonomy consistently with similar test cases.
- Cleanup is complete when all temporary T08 resources are absent.

## Outcome (28 October 2026)

- Technique execution: **Succeeded**. One request reached the controlled health-check Service through the host-allowlisted proxy; HTTP 200 was returned and the response body was discarded.
- Telemetry: Falco was Ready with the modern eBPF driver; the audit receiver recorded `Attach/Exec to pod` about 0.25 seconds after test start. No proxy- or connection-specific syscall alert appeared in the 60-second window.
- Default classification: **Generic**. The observed audit alert identifies the exec access path, not the proxied connection itself.
- Cleanup: Confirmed. The client, proxy pod, ClusterIP Service, and ConfigMap were deleted.
- Evidence: `evidence/T08/T08-forward-proxy-retest-2026-10-28.txt` and local log captures (not for publication).
