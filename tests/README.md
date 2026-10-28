# Controlled Test Procedures

This directory contains individual specifications, preconditions, procedures, and success criteria for the twelve planned techniques. The status table below reflects the evidence-backed results; unrun procedures are plans, not outcomes.

> **Lab-only:** T04–T06 and T11 intentionally exercise node files, sensitive mounts, elevated RBAC, or host networking. Run only in a disposable isolated cluster, and inspect cleanup steps before execution.

## Test Matrix Index

| ID | Tactic | Technique Name | Specification Link | Test Status | Default Result | Retest Status |
|:---:|:---|:---|:---|:---:|:---:|:---:|
| **T01** | Execution | Sidecar injection | [T01-sidecar-injection](T01-sidecar-injection/README.md) | **Completed** | No Detection | **Detected (Custom Rule)** |
| **T02** | Execution | Application exploit (RCE) | [T02-application-rce](T02-application-rce/README.md) | **Completed** | Specific | N/A |
| **T03** | Persistence | Malicious admission controller | [T03-malicious-admission-controller](T03-malicious-admission-controller/README.md) | **Completed** | No Detection | **Detected (Custom Rule)** |
| **T04** | Persistence | Static pods | [T04-static-pods](T04-static-pods/README.md) | **Completed** | **No Detection** | **Detected (Custom Rule)** |
| **T05** | Privilege Escalation | Writable hostPath mount | [T05-writable-hostpath](T05-writable-hostpath/README.md) | **Completed** | Specific | N/A |
| **T06** | Privilege Escalation | Cluster-admin binding | [T06-cluster-admin-binding](T06-cluster-admin-binding/README.md) | **Completed** | Specific | N/A |
| **T07** | Defense Evasion | Delete Kubernetes Events | [T07-delete-kubernetes-events](T07-delete-kubernetes-events/README.md) | **Completed** | No Detection | **Detected (Custom Rule)** |
| **T08** | Defense Evasion | Connect from proxy server | [T08-connect-from-proxy](T08-connect-from-proxy/README.md) | **Completed** | **Generic** | N/A |
| **T09** | Credential Access | Application credentials in configuration files | [T09-application-credentials-in-config](T09-application-credentials-in-config/README.md) | **Completed** | Generic | N/A |
| **T10** | Credential Access | List Kubernetes Secrets | [T10-list-kubernetes-secrets](T10-list-kubernetes-secrets/README.md) | **Completed** | No Detection | N/A |
| **T11** | Discovery | Exposed sensitive interfaces | [T11-exposed-sensitive-interfaces](T11-exposed-sensitive-interfaces/README.md) | **Completed** | Generic | N/A |
| **T12** | Discovery | Network mapping | [T12-network-mapping](T12-network-mapping/README.md) | **Completed** | Generic | N/A |

## Execution Protocol
When conducting a test:
1. Verify the lab environment meets the preconditions documented in [`docs/lab-setup.md`](../docs/lab-setup.md).
2. Record pre-test timestamps and verify telemetry availability.
3. Execute the procedure steps documented in the test's `README.md`.
4. Capture raw output directly into `evidence/T<NN>/`.
5. Classify the outcome according to [`docs/detection-criteria.md`](../docs/detection-criteria.md).
6. Update both the test `README.md` and the master tracker [`results/detection-coverage.csv`](../results/detection-coverage.csv).

All twelve procedures have a recorded result. The generic alert in T08 showed the exec session, not the proxy connection. T12 tested three named service/port pairs; it was a small, bounded check rather than a broad network scan.
