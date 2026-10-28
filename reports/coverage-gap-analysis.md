# Kubernetes Detection Coverage Gap Analysis

## Technical Assessment Report

**Assessment period:** 22–28 October 2026  
**Runtime sensor:** Falco 0.44.1; Helm chart 9.1.0; modern eBPF driver and Kubernetes audit receiver  
**Lab:** Single-node kind environment, Kubernetes 1.37.0  
**Observation window:** 60 seconds after each recorded action  
**Status:** Twelve scenarios attempted; eleven classifiable; T10 baseline remains inconclusive; five custom-rule behaviors retested

## 1. Executive overview

I tested twelve Kubernetes and container scenarios against Falco. This report describes what I saw in the lab configuration listed below; the [evidence register](evidence-register.md) summarizes the supporting captures. The raw captures remain in my local archive and are not part of this repository.

The default configuration produced specific alerts for T05 and T06. Five runs produced context alerts for the `kubectl exec` harness or host-network setup (T02, T08, T09, T11, T12); four produced no relevant default alert (T01, T03, T04, T07). T10's default baseline is inconclusive: its original capture shows a Secret-create alert but not delivery of the list request, and the loaded default rules had no Secret-list rule. A later custom retest confirmed list-event delivery and the new metadata-only rule. Custom retests confirmed alerts for T01, T03, T04, T07, and T10.

Twelve scenarios were attempted; eleven are classifiable. Among those eleven, two (18.2%) produced a specific alert for the behavior under test, five produced context-only alerts, and four had no relevant default alert. T10's original default baseline remains inconclusive, although the later custom retest confirmed list-event delivery. These results apply only to this lab. I did not calculate an overall rate after adding the custom rules because I retested only five scenarios.

## 2. Scope and method

The planned suite covers twelve project-defined techniques. Tests were designed to use isolated lab resources, fake credentials where applicable, and cleanup after execution. Classification follows [the documented criteria](../docs/detection-criteria.md):

| Classification | Rule |
|---|---|
| Specific | An alert identifies the behavior under test and distinguishes it from the test harness or setup. |
| Generic | An alert identifies related activity but not the behavior under test with enough specificity. |
| Context Only | An alert identifies only the mechanism used to run the test or a required setup step. It is not counted as detection. |
| No Detection | The action succeeded, required telemetry was available, and no relevant alert arrived within 60 seconds. |
| Inconclusive | Evidence is insufficient to establish relevant telemetry delivery or whether the matching rule was loaded. |
| Not Run | The scenario was not executed. |

Syscall and Kubernetes audit telemetry are distinct sources. Audit-based detections depend on API-server audit configuration, webhook delivery, and the audit receiver; syscall detections depend on the Falco driver observing the relevant host event. An alert on `kubectl exec` is evidence of the access path, but does not by itself prove that a particular command inside the container was detected by syscall telemetry.

## 3. Environment

| Component | Recorded configuration |
|---|---|
| Kubernetes | kind, version 1.37.0, one control-plane node |
| Falco | 0.44.1, Helm chart 9.1.0 |
| Syscall driver | Modern eBPF |
| API audit | Falco k8saudit receiver and rules artifact |
| Custom rules | T01, T03, T04, T07, and T10 produced alerts in retests. The T10 baseline remains inconclusive; custom behavior is validated only for the recorded test. |

The local cluster was observed healthy during this report update. That check confirms current component readiness only; it does not extend or replace the historical test evidence.

## 4. Results

| ID | Scenario | Default outcome | Evidence-based interpretation | Custom retest |
|---|---|---|---|---|
| T01 | Sidecar injection by workload patch | No Detection | Action succeeded; audit and syscall telemetry were available; no relevant default alert recorded. | **Specific detection observed.** Custom workload-patch alert fired during the 26 Oct retest. |
| T02 | Container command execution via `kubectl exec` (RCE simulation only) | Context Only | `Attach/Exec to pod` audit alert observed at about 1.29 s. This identifies the test harness, not exploitation of an application vulnerability. The separate TTY shell rule was not recorded as firing in this run. | Not applicable. |
| T03 | Admission webhook registration | No Detection | Webhook configuration was created successfully; no default alert recorded with audit telemetry available. | **Specific detection observed.** The published rule without the Kyverno exclusion fired during the 28 Oct retest under the Kyverno service-account identity. |
| T04 | Static pod manifest write | No Detection | Modern eBPF sensor was Ready and the exact manifest `openat` event was observed through the loaded custom rule; no relevant default alert appeared within 60 seconds. | **Custom detection observed** on the same actual manifest write. |
| T05 | Sensitive writable hostPath | Specific | `Create Sensitive Mount Pod` warning observed at about 1.16 s. | Not required. |
| T06 | Cluster-admin binding | Specific | `Attach to cluster-admin Role` warning observed at about 0.57 s. | Not required. |
| T07 | Kubernetes Event deletion | No Detection | Original event deletion succeeded; audit telemetry was available; no default alert recorded. | **Specific detection observed.** A later synthetic Event deletion produced the custom Warning alert; cleanup was verified. |
| T08 | Connect from proxy server | Context Only | One request through a temporary host-allowlisted proxy reached the controlled in-cluster service (HTTP 200; response body discarded). Audit `Attach/Exec to pod` was observed; no proxy- or connection-specific alert appeared in the 60-second window. | Not required. |
| T09 | Application credentials in configuration files | Context Only | Fake credential file and environment values were read. Audit `Attach/Exec to pod` Notice appeared at about 0.34 s; no credential-specific syscall alert was observed. | Not required. |
| T10 | List Kubernetes Secrets | Inconclusive | The original capture shows an adjacent Secret create event, not the list request; the loaded Falco rules had no Secret-list rule. A later retest loaded the project metadata-only rule and observed its alert on a namespaced fake-Secret list. The original baseline remains inconclusive. | **Specific custom alert observed**; no default-rule conclusion. |
| T11 | Exposed sensitive interfaces | Context Only | One request reached the local kubelet endpoint (HTTP 401; body discarded). `Pod started using host network` Warning observed about 0.23 s after test start; it identifies setup only. | Not required. |
| T12 | Network mapping | Context Only | Three bounded TCP probes to named in-cluster Services/ports succeeded. Audit `Attach/Exec to pod` Notice observed about 0.19 s after test start; no network-probe syscall alert was observed. | Not required. |

### 4.1 Baseline summary

- **Specific detections:** 2 (T05, T06)
- **Generic detections:** 0
- **Context-only alerts:** 5 (T02, T08, T09, T11, T12)
- **No Detection:** 4 (T01, T03, T04, T07)
- **Inconclusive:** 1 (T10)
- **Not Run:** 0
- **Classifiable baseline denominator:** 11
- **Technique-specific alert rate:** 2/11 (18.2%)
- **Attempts with any alert, including T10's setup alert:** 8/12 (not a coverage rate)

The four No Detection results are misses for the tested operations under the stated conditions. Context-only alerts in T02, T08, T09, T11, and T12 do not identify the behavior under test. T10 is not included among the misses because event-level telemetry and loaded-rule status were not established.

### 4.2 Latency observations

Recorded alert latencies were approximately 0.57 s (T06), 1.16 s (T05), 1.29 s (T02), 0.25 s (T08), 0.34 s (T09), 0.23 s (T11), and 0.19 s (T12). T02, T08, T09, and T12 alerts identify the exec harness; T11 identifies host-network pod setup. These are individual observations, not a performance benchmark. T04's custom alert latency was not calculated; T01 and T03 retest artifacts show alert timestamps but do not establish a consistently measured end-to-end latency method.

## 5. Gap analysis

### T01 — workload modification

The successful sidecar injection produced no relevant default alert while the audit path was available. The custom rule detects patch/update operations on workload resources, and its retest fired. Because the rule covers a broad set of workload modifications, legitimate controllers may trigger it. Validate and tune exceptions using the actual cluster's controller identities and baseline traffic before production use.

### T03 — admission webhook registration

The successful webhook registration produced no default alert in the recorded baseline. The 26 October retest fired the then-current rule with a Kyverno exclusion. On 28 October, I loaded the published version without that exclusion and created a temporary webhook configuration under the Kyverno admission-controller service-account identity. The rule fired, and cleanup was verified. The rule may alert on legitimate controller changes; tune it in staging before rollout.

### T04 — static pod manifest write

The original run did not validate file-write telemetry. In the 28 October retest, Falco was Ready with modern eBPF, the custom manifest rule was loaded, and a harmless temporary manifest write produced the expected syscall event. No relevant default alert appeared in 60 seconds, so this tested manifest-write path is classified as No Detection. The four original control-plane manifests had unchanged hashes after cleanup. This does not establish that default rules miss every static-pod creation path.

### T08 — connection through an HTTP proxy

The existing application was not a forward proxy, so that attempt was inconclusive. A later retest used a temporary ClusterIP proxy whose Nginx configuration allowed only the controlled in-cluster health-check Service. The request succeeded with HTTP 200 and the body was discarded. Falco emitted an `Attach/Exec to pod` audit alert about 0.25 seconds after the test started, but no alert identifying the proxied connection appeared during the 60-second window. The exec event is Context Only for this network test, not proxy-specific coverage. The client, proxy, Service, and ConfigMap were removed.

### T07 — Kubernetes Event deletion

The baseline event deletion succeeded without a relevant default alert. The first custom retest failed because the event name was empty. A subsequent retest created a uniquely named synthetic Event, deleted it successfully, observed the custom Warning alert within the window, and confirmed the Event was absent afterward. The custom rule is validated for this tested delete path.

### T10 — Kubernetes Secret listing

The original namespaced list succeeded, but its evidence capture contains only a `Secret Created` alert. The loaded default Falco rules did not contain a Secret-list rule, and the capture does not show delivery of the list event; keep that default-baseline result Inconclusive. In a later retest, I loaded the project metadata-only rule, listed a namespaced Secret with synthetic data, observed the custom alert, and removed the Secret. That validates the project rule for this tested path.

## 6. Recommendations

1. **Address the T04 gap.** Consider the validated custom static-manifest rule as a candidate control; review scope, exceptions, and false positives in representative staging.
2. **Review T08 network visibility.** If proxy use must be identified specifically, develop and validate a network-level signal; the current generic exec-session alert does not identify the outbound connection.
3. **Tune T10 before broader use.** The metadata-only custom rule fired on the tested namespaced list. Tune expected controller activity in representative staging; the original default baseline remains inconclusive.
4. **Validate custom rules in staging.** Confirm rule loading, audit policy coverage, receiver availability, and expected controller exceptions for T01/T03/T07/T10. Measure alert latency and false positives using representative traffic.
5. **Monitor audit pipeline health.** Track receiver readiness, webhook delivery errors, event volume, and a benign canary. Alert on delivery failure or unexpected silence.
6. **Retain and publish sanitized evidence.** Keep raw logs and environment exports private. Publish only reviewed summaries that omit identifiers and sensitive values, and retain detailed raw evidence in access-controlled storage.

## 7. Limitations

- All twelve scenarios were attempted. Eleven are classifiable; T10 remains inconclusive because the list event and loaded rule were not verified.
- Supplemental kube-bench, Kubescape, and Trivy baseline artifacts from September 2026 remain local and are not analyzed in this Falco report; their currentness and completeness have not been established.
- Only one cluster topology and one Falco configuration were represented.
- Results use a 60-second window and do not measure long-term reliability or false-positive rates.
- The Falco event timestamp displayed for the T04 syscall retest was offset from the host UTC clock. The exact event was correlated by the unique temporary manifest path and captured from the test log window; no T04 latency is reported.
- Custom-rule retests confirm only the recorded behavior; they do not validate production readiness.
- Raw evidence is excluded from Git. The checked-in evidence register is a sanitized summary, so independent review requires access to the local evidence archive.
- T08 and T12 depend on Kubernetes Goat and `health-check-service` workloads that are not included in this repository; test the named services only in the isolated lab.
- ATT&CK references are descriptive mappings used by this project and do not imply complete technique coverage.

## 8. Conclusion

In these runs, Falco produced two specific detections, five context-only alerts, and no relevant default alert in four scenarios; T10's default result remains inconclusive. Custom-rule alerts were observed for T01, T03, T04, T07, and T10. The T03 rule without the Kyverno exclusion and the T10 metadata-only rule both fired during their retests. The results are limited to this lab: T08's alert shows the exec session, and T12 covered three named in-cluster service/port pairs.

See the [executive summary](executive-summary.md), [evidence register](evidence-register.md), and [CSV tracker](../results/detection-coverage.csv).
