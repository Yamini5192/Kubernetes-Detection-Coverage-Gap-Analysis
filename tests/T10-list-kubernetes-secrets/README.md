# T10-list-kubernetes-secrets

## Objective
Check whether Kubernetes Secret list events reach Falco and validate the project rule for this operation. The loaded Falco 0.44.1 rules observed during the assessment did not include a Secret-list rule, so the existing run cannot establish a default-rule miss without event-level telemetry evidence.

## Preconditions
- Target cluster: `falco-lab` (`kind-falco-lab`).
- Telemetry: Active K8s audit webhook receiver (`falco-audit`).
- A test Secret with fake values must be created before the test to ensure there is a resource to list.
- Use only fake secret values. No real credentials are to be used at any point.
- Record the test start timestamp before issuing the list command.

## Procedure
1. Record UTC test start time.
2. Create a test Secret with fake values in the `falco-research` namespace:
   ```bash
   kubectl create secret generic t10-test-secret \
     --from-literal=username=fake-user \
     --from-literal=password=fake-secret-value-123 \
     -n falco-research
   ```
3. Issue a `list` verb request against the secrets API in the `falco-research` namespace:
   ```bash
   kubectl get secrets -n falco-research
   ```
4. Also issue a cluster-wide secrets list to test broader coverage:
   ```bash
   kubectl get secrets --all-namespaces
   ```
5. Monitor Falco k8saudit logs for the 60-second detection window from the first kubectl get timestamp. Look specifically for:
   - Confirm from the loaded audit rules that `Project T10 Kubernetes Secrets Listed` is loaded and matches `ka.verb = list` with `ka.target.resource = secrets`.
   - Confirm that the list request itself appears in the delivered audit event stream. A nearby Secret create event only confirms delivery of that create event.
6. Record all observed alerts, rule names, priorities, and calculate latency from the timestamp of the kubectl get command.
7. Delete the test Secret after the detection window closes.

## Success criteria
- Technique execution: The `kubectl get secrets` command returns a list of secrets from the API server.
- Telemetry: The list request appears in the audit event stream and the matching rule is loaded.
- Detection: If those checks pass, record whether `T10_KUBERNETES_SECRETS_LISTED` arrives within 60 seconds. This validates the project custom rule, not the default Falco ruleset.
- Cleanup: The test Secret `t10-test-secret` is deleted.

## Evidence
- Evidence log: `evidence/T10/T10-list-kubernetes-secrets-2026-10-28.txt`
  - Pre-test state: Falco health check and loaded-rule confirmation for `Project T10 Kubernetes Secrets Listed`
  - Test execution: kubectl create secret output, kubectl get secrets output (both namespaced and cluster-wide)
  - Detection window: Falco log extract (60 seconds from kubectl get timestamp), showing rule name, priority, and latency
  - Cleanup confirmation: kubectl delete secret output

## Outcome
- Baseline status: Attempted (28 October 2026); classification remains Inconclusive.
- Technique execution: Succeeded. A namespaced list returned the temporary test Secret; secret contents were not requested or displayed.
- Telemetry availability: The original capture shows an adjacent Secret create alert, not the list request. A later custom-rule retest confirmed the list event reached Falco. The loaded default rules did not include a Secret-list rule.
- Default alert: No list alert appeared in the captured Falco logs; no Secret-list rule was loaded, and event-level delivery was not documented.
- Classification: **Inconclusive**. Do not treat this run as a confirmed detection gap.
- Custom-rule retest: **Detected**. The loaded `Project T10 Kubernetes Secrets Listed` rule emitted `T10_KUBERNETES_SECRETS_LISTED` for the namespaced list request. The rule output contains request metadata only.
- Cleanup: Confirmed. The fake test Secret was deleted.
- Evidence: `evidence/T10/T10-list-kubernetes-secrets-2026-10-28.txt` (local archive).
- Custom retest evidence: `evidence/T10/T10-project-rule-retest-2026-10-28.txt` (local archive).
