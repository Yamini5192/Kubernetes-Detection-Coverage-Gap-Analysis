# GitHub Safe Release Package

This public repository contains the project files selected for release. I left out the original local Git history, raw test evidence, baseline screenshots and logs, backups, kubeconfig, credentials, and private keys. The evidence register contains sanitized summaries, alongside the current result tracker.

## Assessment status

I ran and classified all twelve scenarios: three produced specific detections, four produced generic detections, and five had no relevant default alert. The observed relevant-alert rate is 7/12 (58.3%). T08's alert shows an exec session, not the proxy connection. T12 tested three named in-cluster services and ports; it was not a broad scan. These results describe this lab assessment only.

Supplemental September 2026 kube-bench, Kubescape, and Trivy baseline captures were reviewed for publication sensitivity but are not incorporated into the Falco analysis; their currentness and completeness are not established. Raw baseline material is deliberately absent.

The current T03 rule source is broader than the version exercised in its retest because it removes a Kyverno identity exclusion. Validate that expanded rule before deployment.

## Publication safety

1. Do not copy local `baseline/`, `evidence/`, or `backups/` contents into this repository.
2. Review and rescan any new files before committing them.
3. No license is included. Add one only after choosing the reuse terms you want.

I checked the release files for common credential and private-key patterns. That check is useful as a precaution, but it cannot guarantee that every sensitive value has been found.

The included audit-receiver Helm example exposes a NodePort over plain HTTP without receiver authentication. It is intentionally lab-only and should not be deployed to shared or production clusters as-is.
