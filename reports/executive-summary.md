# Executive Summary

**Assessment:** Kubernetes runtime detection coverage using Falco 0.44.1  
**Assessment period:** 22–28 October 2026  
**Environment:** Single-node kind lab; Kubernetes 1.37.0; Falco Helm chart 9.1.0  
**Audience:** Security and platform engineering leadership

## Decision summary

All twelve planned scenarios have recorded, classifiable outcomes. Falco's default configuration generated a relevant alert in seven and produced no relevant alert in five within the defined 60-second window. T04's validated manifest-write event was observed by the custom syscall rule, but no default rule alert appeared. T08 succeeded through a restricted in-cluster proxy; its generic audit alert identified `kubectl exec`, not the proxy connection.

Custom-rule retests observed alerts for workload-spec patching (T01), admission webhook registration (T03), Event deletion (T07), and T04 manifest writes. The T07 first retest failed because no event name was supplied, but a later successful retest produced the expected alert. The published T03 rule removes the Kyverno exclusion used in its retest; this expanded variant still needs validation. No overall post-custom-rule coverage percentage is reported because custom retests covered only a subset of scenarios.

## Results at a glance

| Measure | Result |
|---|---:|
| Planned scenarios | 12 |
| Scenarios attempted | 12 |
| Default detections | 7 (T02, T05, T06 specific; T08, T09, T11, T12 generic) |
| Confirmed default misses | 5 (T01, T03, T04, T07, T10) |
| Inconclusive | 0 |
| Confirmed custom-rule retests | 4 (T01, T03, T04, T07) |
| Not run | 0 |

Default relevant-alert rate among classifiable tests: **7/12 (58.3%)**. This is a lab result for the tested scenarios and configuration, not a general Falco effectiveness estimate. T08, T09, and T12 generic exec alerts identify access paths, not the proxied connection, credential reading, or network mapping. T11's generic host-network alert identifies the setup, not the kubelet request.

## Key findings

1. **Control-plane and credential discovery monitoring have verified gaps.** Default Falco produced no relevant alert for sidecar injection (T01), admission webhook registration (T03), the tested static-manifest write (T04), Kubernetes Event deletion (T07), or listing Kubernetes Secrets (T10), despite successful actions and the relevant telemetry checks described in the technical report.
2. **High-impact activity was detected in the tested cases.** Falco emitted specific audit alerts for creation of a sensitive hostPath pod (T05) and a cluster-admin binding (T06). Generic alerts identified the exec path for T02, T08, T09, and T12, and host-network pod setup for T11.
3. **Custom rule validation is partial.** T01, T03, T04, and T07 fired during retests. The T04 custom rule observed the validated syscall, while default rules produced no relevant alert for that write.
4. **Network-specific visibility remains a gap.** T08's proxy request and T12's three bounded TCP probes succeeded, but Falco only identified the exec paths, not the network activity. These tests do not establish coverage for broader network reconnaissance.

## Priority actions

| Priority | Action | Rationale |
|---|---|---|
| 1 | Review the T04 custom static-manifest rule in representative staging and tune its scope and exceptions. | The rule detected the tested write; production readiness and false-positive rate are not established. |
| 2 | Define whether T08 requires a proxy-specific network alert and validate an appropriate signal. | The current generic exec alert does not identify the proxied connection. |
| 3 | Review the T10 gap and define a detection strategy for Secret list operations appropriate to the environment. | No list alert appeared in the successful test. |
| 4 | Deploy and validate T01/T03/T07 rules in representative staging; tune exceptions against known controllers. | Lab retests confirm rule behavior, not production readiness or false-positive rate. |
| 5 | Monitor audit receiver health and route alerts to durable storage/SIEM; expand network visibility only with an approved, bounded target set if broader reconnaissance coverage is required. | Audit-based findings depend on continuous delivery; the T08/T12 alerts did not identify network activity. |

## Interpretation and limitations

The assessment uses a 60-second observation window and a specific/generic/no-detection/inconclusive/not-run taxonomy. T12 used three named in-cluster services/ports and was not a comprehensive scan. Supplemental kube-bench, Kubescape, and Trivy baseline captures exist locally from September 2026, but they have not been reviewed for currentness or synthesized into this Falco report. Evidence files and raw environment captures remain local and are excluded from version control. The checked-in evidence register contains sanitized summaries; readers cannot independently reproduce raw observations from the repository alone. False-positive rates, production performance, resilience under load, and coverage outside these exact scenarios were not measured.

See the [technical report](coverage-gap-analysis.md), [evidence register](evidence-register.md), and [coverage tracker](../results/detection-coverage.csv) for details.
