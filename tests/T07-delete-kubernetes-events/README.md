# T07-delete-kubernetes-events

## Objective
Evaluate how effectively Falco's default rules detect the deletion of Kubernetes events to conceal malicious activity.

## Preconditions
- Target: `falco-lab` (`kind-falco-lab`).
- Telemetry: Active K8s audit webhook receiver (`falco-audit`).

## Procedure
1. Record UTC test start time.
2. Identify an existing event in the cluster.
3. Issue a delete request for the event via `kubectl delete event <event-name>`.
4. Monitor Falco audit logs for alerts within the 60-second window.
5. Record observed alerts, rule names, and calculate latency.

## Success criteria
- Technique execution: The event is deleted and removed from the API server.
- Detection: Falco triggers an alert identifying per-object deletion of Kubernetes Event API objects. Collection deletion is not covered by the current rule.

## Evidence
- Evidence log (local archive): `evidence/T07/T07-delete-kubernetes-events-2026-10-22.txt`
- Custom-rule retest attempt: `evidence/T07/T07-custom-rule-retest-2026-10-26.txt`
- Successful custom-rule retest: `evidence/T07/T07-custom-rule-retest-2026-10-28.txt`

## Outcome
- Test status: Completed
- Technique execution: Succeeded (event deleted successfully).
- Telemetry availability: Available (K8s API audit stream active).
- Default alert: None observed in captured logs.
- Classification: **No Detection** (Substantiated detection gap).
- Custom-rule retest: **Detected**. A separate synthetic Event was deleted successfully; `T07_KUBERNETES_EVENT_DELETED` emitted a Warning alert in the observation window. The earlier failed attempt remains documented but was superseded by this successful retest.
