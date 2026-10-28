# Executive Summary

**Assessment:** Kubernetes runtime detection coverage using Falco 0.44.1  
**Assessment period:** 22–28 October 2026  
**Environment:** Single-node kind lab; Kubernetes 1.37.0; Falco Helm chart 9.1.0  
**Audience:** Security and platform engineering leadership

## Decision summary

I ran all twelve scenarios and classified each one. Falco's default configuration produced a relevant alert in seven cases; in five cases, no relevant alert appeared within 60 seconds. For T04, the custom syscall rule saw the manifest write, while the default rules did not alert. T08 reached the controlled service through a restricted in-cluster proxy, but Falco's generic audit alert showed only the `kubectl exec` session.

I also retested custom rules for workload patching (T01), webhook registration (T03), Event deletion (T07), and manifest writes (T04). The first T07 retest did not run because I had not supplied an Event name; a later run succeeded and generated the alert. The T03 rule in this repository no longer excludes Kyverno, but I have not retested that broader version. Since the custom rules were tested against only four scenarios, I have not calculated an overall post-custom-rule rate.

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

Falco generated a relevant alert in **7 of 12 scenarios (58.3%)**. This describes these tests and this lab setup only. The generic alerts for T08, T09, and T12 show an exec session; they do not identify the proxy connection, credential access, or network probes. T11's alert shows that the pod used host networking, not that a kubelet request was made.

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

I used a 60-second observation window and classified results as specific, generic, no detection, inconclusive, or not run. T12 checked three named services and ports, not a comprehensive scan. I have older kube-bench, Kubescape, and Trivy captures from September 2026, but I have not checked whether they are current or included them in this Falco report. Raw captures stay local; the evidence register contains sanitized summaries, so this repository alone is not enough to reproduce every observation. I did not measure false-positive rates, production performance, behavior under load, or coverage beyond these scenarios.

See the [technical report](coverage-gap-analysis.md), [evidence register](evidence-register.md), and [coverage tracker](../results/detection-coverage.csv) for details.
