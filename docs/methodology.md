# Methodology

## Research question
Which of the twelve selected Kubernetes/container behaviors generate specific detections, context-only alerts, no detection, or an inconclusive result under the tested default Falco configuration? Which confirmed gaps can be addressed with custom rules?

## Experimental controls
- Target only the `falco-lab` cluster.
- Run one technique at a time and record its start/end time.
- Use harmless lab workloads, fake credentials, and narrowly scoped resources.
- Do not access host secrets or real credentials.
- Preserve existing clusters and workloads.
- Record failures and telemetry limitations instead of interpreting them as successful tests.

## Per-test evidence
Record the technique ID, prerequisites, exact test steps, success criteria, attack timestamp, Falco alert/rule/priority, alert latency, supporting evidence, and test outcome.

## Comparison
Compare the default-rule result with the custom-rule retest using the same success criteria and 60-second detection window. Document configuration changes between runs.
