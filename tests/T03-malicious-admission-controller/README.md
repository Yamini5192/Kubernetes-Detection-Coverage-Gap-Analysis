# T03-malicious-admission-controller

## Objective
Evaluate how effectively Falco's default rules detect the registration of an unauthorized admission controller / validating webhook in Kubernetes.

## Preconditions
- Target: `falco-lab` (`kind-falco-lab`).
- Telemetry: Active K8s audit webhook receiver (`falco-audit`) and syscall engine (`falco`).

## Procedure
1. Record UTC test start time.
2. Apply a test `ValidatingWebhookConfiguration` resource (`t03-test-webhook`) configured with `failurePolicy: Ignore` and target URL.
3. Monitor Falco audit and syscall logs for alerts within the 60-second window.
4. Record observed alerts, rule names, and calculate latency.
5. Delete the test webhook resource immediately.

## Success criteria
- Technique execution: The webhook configuration is accepted by the API server.
- Detection: An alert is triggered identifying the creation of an admission webhook.
- Cleanup: The webhook configuration is removed without affecting existing cluster webhooks.

## Evidence
- Evidence log (local archive): `evidence/T03/T03-malicious-admission-controller-2026-10-22.txt`
- Custom-rule retest: `evidence/T03/T03-custom-rule-retest-2026-10-26.txt`

The retest used the then-current rule, which excluded Kyverno service accounts. The published rule removes that broad exclusion; this expanded variant has not been redeployed/retested and should be validated before rollout.

## Outcome
- Test status: Completed
- Technique execution: Succeeded (`validatingwebhookconfiguration.admissionregistration.k8s.io/t03-test-webhook` created).
- Telemetry availability: Available (K8s API audit stream and syscall driver verified active).
- Default alert: None observed in captured logs.
- Classification: **No Detection** (Substantiated detection gap).
- Custom-rule retest: **Detected**. The T03 audit rule emitted Warning alerts during successful webhook configuration creation.
- Cleanup: Confirmed. Resource deleted.
