# Executive Summary

**Assessment:** Kubernetes runtime detection coverage using Falco 0.44.1  
**Assessment period:** 22–28 October 2026  
**Environment:** Single-node kind lab; Kubernetes 1.37.0; Falco Helm chart 9.1.0  
**Audience:** Security and platform engineering leadership

## Decision summary

I attempted all twelve scenarios. Two classifiable runs produced specific alerts for the behavior under test (T05 and T06). Five produced context alerts for the `kubectl exec` harness or host-network setup, and four produced no relevant default alert. T10 is inconclusive: its capture shows a Secret create event, but not whether the list request reached Falco or whether the matching rule was loaded. For T04, the custom syscall rule saw the manifest write, but the default rules did not alert. T08 reached the controlled service through the restricted proxy; its audit alert showed only the exec session.

I also retested custom rules for workload patching (T01), webhook registration (T03), Event deletion (T07), manifest writes (T04), and Secret listing (T10). The first T07 retest did not run because I had not supplied an Event name; a later run succeeded and generated the alert. The published T03 version without a Kyverno exclusion fired when tested under the Kyverno admission-controller service-account identity. The T10 metadata-only rule fired on a namespaced list of a fake Secret. Since custom rules were tested against only five scenarios, I have not calculated an overall post-custom-rule rate.

## Results at a glance

| Measure | Result |
|---|---:|
| Planned scenarios | 12 |
| Scenarios attempted | 12 |
| Specific detections | 2 (T05, T06) |
| Context-only alerts | 5 (T02, T08, T09, T11, T12) |
| No Detection | 4 (T01, T03, T04, T07) |
| Inconclusive | 1 (T10) |
| Confirmed custom-rule retests | 5 (T01, T03, T04, T07, T10) |
| Not run | 0 |

Only **2 of the 11 classifiable scenarios (18.2%)** produced an alert that identified the behavior under test. Five other classifiable runs produced alerts for the exec harness or host-network setup. T10 also produced a Secret-create setup alert, but the capture does not show whether the Secret-list event reached Falco. T08's alert does not identify the proxy connection; T09's does not identify credential access; T11's does not identify the kubelet request; and T12's does not identify network probes.

## Key findings

1. **Four tested operations had no relevant default alert.** These were sidecar injection (T01), admission webhook registration (T03), the tested static-manifest write (T04), and Kubernetes Event deletion (T07). T10 is inconclusive because the list event and loaded rule were not verified.
2. **Two tested behaviors were specifically detected.** Falco emitted specific audit alerts for creation of a sensitive hostPath pod (T05) and a cluster-admin binding (T06). T02 used `kubectl exec` to simulate command execution, so its `Attach/Exec to pod` alert identifies the test mechanism, not RCE; it is classified Context Only.
3. **Custom rule validation is partial.** T01, T03, T04, T07, and T10 fired during retests. The T04 custom rule observed the validated syscall, while default rules produced no relevant alert for that write. The T10 baseline remains inconclusive even though the custom rule retest confirmed list-event delivery.
4. **Network-specific visibility remains a gap.** T08's proxy request and T12's three bounded TCP probes succeeded, but Falco only logged the exec harness, not the network activity. These tests do not establish coverage for broader network reconnaissance.

## Priority actions

| Priority | Action | Rationale |
|---|---|---|
| 1 | Review the T04 custom static-manifest rule in representative staging and tune its scope and exceptions. | The rule detected the tested write; production readiness and false-positive rate are not established. |
| 2 | Define whether T08 requires a proxy-specific network alert and validate an appropriate signal. | The current context-only exec alert does not identify the proxied connection. |
| 3 | Tune the T10 rule against expected controller activity in representative staging. | The metadata-only rule fired on the tested list; production readiness and false-positive rates are not established. |
| 4 | Deploy and validate T01/T03/T07 rules in representative staging; tune exceptions against known controllers. | Lab retests confirm rule behavior, not production readiness or false-positive rate. |
| 5 | Monitor audit receiver health and route alerts to durable storage/SIEM; expand network visibility only with an approved, bounded target set if broader reconnaissance coverage is required. | Audit-based findings depend on continuous delivery; the T08/T12 alerts did not identify network activity. |

## Interpretation and limitations

I used a 60-second observation window and classified results as specific, generic, context only, no detection, inconclusive, or not run. T10's capture does not establish delivery of the Secret list event or confirm that the matching rule was loaded, so I have not called it a detection gap. T12 checked three named services and ports, not a comprehensive scan. I have older kube-bench, Kubescape, and Trivy captures from September 2026, but I have not checked whether they are current or included them in this Falco report. Raw captures stay local; the evidence register contains sanitized summaries, so this repository alone is not enough to reproduce every observation. I did not measure false-positive rates, production performance, behavior under load, or coverage beyond these scenarios.

See the [technical report](coverage-gap-analysis.md), [evidence register](evidence-register.md), and [coverage tracker](../results/detection-coverage.csv) for details.
