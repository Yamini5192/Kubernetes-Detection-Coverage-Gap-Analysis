# Custom Falco Rules

This directory contains project-authored rules for behavior observed during the assessment. Rule presence or successful loading does not prove detection; only a successful, evidence-backed retest supports that conclusion.

## Inventory and validation status

| Rule | Source | Status |
|---|---|---|
| T01 workload spec patched | Kubernetes audit | Retest alert observed on 26 October 2026. Matches workload patch/update operations broadly; tune approved controller identities and expected automation in staging. |
| T03 admission webhook configuration created or updated | Kubernetes audit | Retest alert observed on 26 October 2026 with a Kyverno service-account exclusion. The published source removes that broad exclusion; this expanded variant has not been redeployed/retested and may alert on legitimate controller changes. Validate before rollout. |
| T04 static pod manifest opened for write | Syscall | Validated retest observed the actual manifest-write syscall; no default-rule alert appeared for that tested path. |
| T07 Kubernetes Event deleted | Kubernetes audit | Successful per-object delete retest observed on 28 October 2026. Collection deletion (`deletecollection`) is not currently matched or validated. |

Definitions are maintained in the rule YAML files and, for audit rules, in `helm/falco-audit-values.yaml`. The `project-custom-test.yaml` smoke-test rule remains a diagnostic artifact and should not be treated as a production detection. ATT&CK references are included only where the mapping fits the observed behavior; T03/T04/T07 remain project-defined behavior labels without a direct technique ID.

## Deployment configuration

- Runtime syscall rule T04: [`helm/falco-release-values.yaml`](../helm/falco-release-values.yaml)
- Audit rules T01, T03, and T07: [`helm/falco-audit-values.yaml`](../helm/falco-audit-values.yaml)

See the [technical report](../reports/coverage-gap-analysis.md) for limitations and rollout recommendations. Test rules in staging, verify the audit policy and rule loading, and tune allowlists against observed controller traffic before production use.
