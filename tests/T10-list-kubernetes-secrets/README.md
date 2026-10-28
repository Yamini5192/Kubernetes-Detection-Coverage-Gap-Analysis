# T10-list-kubernetes-secrets

## Objective
Evaluate how effectively Falco's default Kubernetes API audit rules detect the listing of Kubernetes Secret resources, a common initial step in credential harvesting attacks.

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
   - `List Kubernetes Secrets` (default k8saudit rule: fires when `ka.verb = list` and `ka.target.resource = secrets`)
6. Record all observed alerts, rule names, priorities, and calculate latency from the timestamp of the kubectl get command.
7. Delete the test Secret after the detection window closes.

## Success criteria
- Technique execution: The `kubectl get secrets` command returns a list of secrets from the API server.
- Detection: The Falco k8saudit rule `List Kubernetes Secrets` fires within 60 seconds of the list request, at the expected priority level.
- Cleanup: The test Secret `t10-test-secret` is deleted.

## Evidence
- Evidence log: `evidence/T10/T10-list-kubernetes-secrets-<date>.txt`
  - Pre-test state: secrets list, Falco health check and active rule confirmation for `List Kubernetes Secrets`
  - Test execution: kubectl create secret output, kubectl get secrets output (both namespaced and cluster-wide)
  - Detection window: Falco log extract (60 seconds from kubectl get timestamp), showing rule name, priority, and latency
  - Cleanup confirmation: kubectl delete secret output

## Outcome
- Test status: Completed (28 October 2026)
- Technique execution: Succeeded. A namespaced list returned the temporary test Secret; secret contents were not requested or displayed.
- Telemetry availability: Kubernetes audit receiver active; adjacent Secret create activity appeared in the audit alert stream.
- Default alert: No relevant list alert observed in the 60-second window.
- Classification: **No Detection** under the tested configuration.
- Cleanup: Confirmed. The fake test Secret was deleted.
- Evidence: `evidence/T10/T10-list-kubernetes-secrets-2026-10-28.txt` (local archive).
