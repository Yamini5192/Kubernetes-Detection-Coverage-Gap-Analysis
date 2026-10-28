# Kubernetes Detection Coverage Gap Analysis

## Technical Assessment Report

**Assessment period:** 22–28 October 2026  
**Runtime sensor:** Falco 0.44.1; Helm chart 9.1.0; modern eBPF driver and Kubernetes audit receiver  
**Lab:** Single-node kind environment, Kubernetes 1.37.0  
**Observation window:** 60 seconds after each recorded action  
**Status:** All twelve planned scenarios attempted and classified; four custom-rule retests confirmed

## 1. Executive overview

This report evaluates observed Falco detections for twelve planned Kubernetes/container scenarios. Results are limited to the recorded lab configuration and evidence summarized in the [evidence register](evidence-register.md). Raw captures are kept locally and excluded from version control.

The default configuration produced specific alerts for T02, T05, and T06; generic alerts for T08, T09, T11, and T12; and no relevant alert for T01, T03, T04, T07, and T10 under the recorded test conditions. T04's retest independently verified the manifest-write syscall event through the custom rule while no default alert appeared. T08's restricted proxy request succeeded; its generic alert identified the `kubectl exec` access path, not the proxied network connection. Custom retests confirmed alerts for T01, T03, T04, and T07.

All twelve scenarios are classifiable. The observed relevant-alert rate was 7/12 (58.3%), including four generic alerts. This figure describes only these scenarios and this lab configuration; it should not be generalized to other clusters or Falco configurations. No overall post-custom-rule coverage rate is calculated because custom retests covered only a subset of scenarios.

## 2. Scope and method

The planned suite covers twelve project-defined techniques. Tests were designed to use isolated lab resources, fake credentials where applicable, and cleanup after execution. Classification follows [the documented criteria](../docs/detection-criteria.md):

| Classification | Rule |
|---|---|
| Specific | A relevant alert precisely identifies the technique or behavior. |
| Generic | A relevant but less-specific alert occurs within 60 seconds. |
| No Detection | The action succeeded, required telemetry was available, and no relevant alert arrived within 60 seconds. |
| Inconclusive | The action failed, evidence is insufficient, or relevant telemetry was not established. |
| Not Run | The scenario was not executed. |

Syscall and Kubernetes audit telemetry are distinct sources. Audit-based detections depend on API-server audit configuration, webhook delivery, and the audit receiver; syscall detections depend on the Falco driver observing the relevant host event. An alert on `kubectl exec` is evidence of the access path, but does not by itself prove that a particular command inside the container was detected by syscall telemetry.

## 3. Environment

| Component | Recorded configuration |
|---|---|
| Kubernetes | kind, version 1.37.0, one control-plane node |
| Falco | 0.44.1, Helm chart 9.1.0 |
| Syscall driver | Modern eBPF |
| API audit | Falco k8saudit receiver and rules artifact |
| Custom rules | T01, T03, T04, and T07 authored; successful retest alerts observed for all four |

The local cluster was observed healthy during this report update. That check confirms current component readiness only; it does not extend or replace the historical test evidence.

## 4. Results

| ID | Scenario | Default outcome | Evidence-based interpretation | Custom retest |
|---|---|---|---|---|
| T01 | Sidecar injection by workload patch | No Detection | Action succeeded; audit and syscall telemetry were available; no relevant default alert recorded. | **Specific detection observed.** Custom workload-patch alert fired during the 26 Oct retest. |
| T02 | Application command execution / RCE simulation | Specific | `Attach/Exec to pod` audit alert observed at about 1.29 s. This identifies the exec access path; complementary syscall shell detection was noted for interactive TTY use. | Not required. |
| T03 | Admission webhook registration | No Detection | Webhook configuration was created successfully; no default alert recorded with audit telemetry available. | **Specific detection observed.** Custom webhook-configuration alert fired during the 26 Oct retest. |
| T04 | Static pod manifest write | No Detection | Modern eBPF sensor was Ready and the exact manifest `openat` event was observed through the loaded custom rule; no relevant default alert appeared within 60 seconds. | **Custom detection observed** on the same actual manifest write. |
| T05 | Sensitive writable hostPath | Specific | `Create Sensitive Mount Pod` warning observed at about 1.16 s. | Not required. |
| T06 | Cluster-admin binding | Specific | `Attach to cluster-admin Role` warning observed at about 0.57 s. | Not required. |
| T07 | Kubernetes Event deletion | No Detection | Original event deletion succeeded; audit telemetry was available; no default alert recorded. | **Specific detection observed.** A later synthetic Event deletion produced the custom Warning alert; cleanup was verified. |
| T08 | Connect from proxy server | Generic | One request through a temporary host-allowlisted proxy reached the controlled in-cluster service (HTTP 200; response body discarded). Audit `Attach/Exec to pod` was observed; no proxy- or connection-specific syscall alert appeared in the 60-second window. | Not required. |
| T09 | Application credentials in configuration files | Generic | Fake credential file and environment values were read. Audit `Attach/Exec to pod` Notice appeared at about 0.34 s; no credential-specific syscall alert was observed. | Not required. |
| T10 | List Kubernetes Secrets | No Detection | Namespaced list succeeded with audit receiver active; no relevant list alert appeared in the 60-second window. | Not required. |
| T11 | Exposed sensitive interfaces | Generic | One request reached the local kubelet endpoint (HTTP 401; body discarded). `Pod started using host network` Warning observed about 0.23 s after test start. | Not required. |
| T12 | Network mapping | Generic | Three bounded TCP probes to named in-cluster Services/ports succeeded. Audit `Attach/Exec to pod` Notice observed at about 0.19 s; no network-probe syscall alert was observed. | Not required. |

### 4.1 Baseline summary

- **Specific detections:** 3 (T02, T05, T06)
- **Generic detections:** 4 (T08, T09, T11, T12)
- **No Detection:** 5 (T01, T03, T04, T07, T10)
- **Inconclusive:** 0
- **Not Run:** 0
- **Classifiable baseline denominator:** 12
- **Observed relevant alerts among classifiable runs:** 7/12 (58.3%)

The five No Detection results are misses for the tested operations under the stated conditions. T08's generic exec alert does not identify proxy use or the outbound connection. T09's and T12's generic exec alerts do not specifically identify credential access or network mapping; T11's host-network alert identifies the setup, not the kubelet request.

### 4.2 Latency observations

Recorded default alert latencies were approximately 0.57 s (T06), 1.16 s (T05), 1.29 s (T02), 0.25 s (T08), 0.34 s (T09), 0.23 s (T11), and 0.19 s (T12). For T08, T09, and T12, the measured alert identifies the exec access path; for T11, it identifies host-network pod setup. These are individual observations, not a performance benchmark. T04's custom alert latency was not calculated; T01 and T03 retest artifacts show alert timestamps but do not establish a consistently measured end-to-end latency method.

## 5. Gap analysis

### T01 — workload modification

The successful sidecar injection produced no relevant default alert while the audit path was available. The custom rule detects patch/update operations on workload resources, and its retest fired. Because the rule covers a broad set of workload modifications, legitimate controllers may trigger it. Validate and tune exceptions using the actual cluster's controller identities and baseline traffic before production use.

### T03 — admission webhook registration

The successful webhook registration produced no default alert in the recorded baseline. A retest fired the then-current custom rule on a test `ValidatingWebhookConfiguration` creation. The published rule now alerts on create/update/patch operations without identity exclusions; that expanded variant has not been redeployed/retested and may alert on legitimate controller changes. Validate both validating and mutating webhook resource types in staging.

### T04 — static pod manifest write

The original run did not validate file-write telemetry. In the 28 October retest, Falco was Ready with modern eBPF, the custom manifest rule was loaded, and a harmless temporary manifest write produced the expected syscall event. No relevant default alert appeared in 60 seconds, so this tested manifest-write path is classified as No Detection. The four original control-plane manifests had unchanged hashes after cleanup. This does not establish that default rules miss every static-pod creation path.

### T08 — connection through an HTTP proxy

The existing application was not a forward proxy, so that attempt was inconclusive. A later retest used a temporary ClusterIP proxy whose Nginx configuration allowed only the controlled in-cluster health-check Service. The request succeeded with HTTP 200 and the body was discarded. Falco emitted a generic `Attach/Exec to pod` audit alert about 0.25 seconds after the test started, but no alert identifying the proxied connection appeared during the 60-second window. This is a generic detection of the access path, not proxy-specific network coverage. The client, proxy, Service, and ConfigMap were removed.

### T07 — Kubernetes Event deletion

The baseline event deletion succeeded without a relevant default alert. The first custom retest failed because the event name was empty. A subsequent retest created a uniquely named synthetic Event, deleted it successfully, observed the custom Warning alert within the window, and confirmed the Event was absent afterward. The custom rule is validated for this tested delete path.

### T10 — Kubernetes Secret listing

The namespaced Secret list succeeded using a temporary Secret with fake data. The audit receiver was active, but no relevant `List Kubernetes Secrets` alert appeared in the 60-second window. The test Secret was removed. This is a confirmed miss for the tested list operation and the recorded rule configuration.

## 6. Recommendations

1. **Address the T04 gap.** Consider the validated custom static-manifest rule as a candidate control; review scope, exceptions, and false positives in representative staging.
2. **Review T08 network visibility.** If proxy use must be identified specifically, develop and validate a network-level signal; the current generic exec-session alert does not identify the outbound connection.
3. **Review T10 detection strategy.** Decide whether and how to alert on Secret list operations, then tune for expected API clients and validate the rule in staging.
4. **Validate custom rules in staging.** Confirm rule loading, audit policy coverage, receiver availability, and expected controller exceptions for T01/T03/T07. Measure alert latency and false positives using representative traffic.
5. **Monitor audit pipeline health.** Track receiver readiness, webhook delivery errors, event volume, and a benign canary. Alert on delivery failure or unexpected silence.
6. **Retain and publish sanitized evidence.** Keep raw logs and environment exports private. Publish only reviewed summaries that omit identifiers and sensitive values, and retain detailed raw evidence in access-controlled storage.

## 7. Limitations

- All twelve planned scenarios were attempted and classified; no scenario remains unrun or inconclusive.
- Supplemental kube-bench, Kubescape, and Trivy baseline artifacts from September 2026 remain local and are not analyzed in this Falco report; their currentness and completeness have not been established.
- Only one cluster topology and one Falco configuration were represented.
- Results use a 60-second window and do not measure long-term reliability or false-positive rates.
- The Falco event timestamp displayed for the T04 syscall retest was offset from the host UTC clock. The exact event was correlated by the unique temporary manifest path and captured from the test log window; no T04 latency is reported.
- Custom-rule retests confirm only the recorded behavior; they do not validate production readiness.
- Raw evidence is excluded from Git. The checked-in evidence register is a sanitized summary, so independent review requires access to the local evidence archive.
- ATT&CK references are descriptive mappings used by this project and do not imply complete technique coverage.

## 8. Conclusion

The available evidence supports three specific detections, four generic detections, and five default misses across twelve attempted scenarios. Custom retests confirm T01, T03, T04, and T07 behavior. All planned scenarios now have classifiable results, but the assessment remains a bounded lab measurement: the T08 alert identifies only the exec access path, and T12 tested only three explicitly named in-cluster services/ports.

See the [executive summary](executive-summary.md), [evidence register](evidence-register.md), and [CSV tracker](../results/detection-coverage.csv).
