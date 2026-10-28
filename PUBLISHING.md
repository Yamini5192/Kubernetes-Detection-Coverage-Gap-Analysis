# GitHub Safe Release Package

This is a fresh-source package prepared from the reviewed project files. It contains no `.git` history, raw test evidence, baseline screenshots/logs, backups, kubeconfig, credentials, or private keys. It includes the sanitized evidence register and the current result tracker.

## Assessment status

All twelve scenarios have a recorded, classifiable outcome: three specific detections, four generic detections, and five no-detection results. The observed relevant-alert rate is 7/12 (58.3%). T08's generic exec alert does not identify proxy use or the outbound connection. T12 tested three named in-cluster services/ports, not a comprehensive scan. This is a bounded lab assessment, not a comprehensive security certification.

Supplemental September 2026 kube-bench, Kubescape, and Trivy baseline captures were reviewed for publication sensitivity but are not incorporated into the Falco analysis; their currentness and completeness are not established. Raw baseline material is deliberately absent.

The current T03 rule source is broader than the version exercised in its retest because it removes a Kyverno identity exclusion. Validate that expanded rule before deployment.

## Before publishing

1. Review the sanitized report and tracker for intended audience and accuracy.
2. Choose and add a license if you want others to reuse the project; none is supplied.
3. Publish this folder as a new repository or copy its contents into a clean repository. It intentionally has no prior Git history.
4. Do not add local `baseline/`, `evidence/`, or `backups/` contents. They are explicitly excluded from this package.

This package was assembled from an explicit file allowlist. It was scanned for common credential/private-key patterns; that check is a precaution and does not prove absence of every sensitive value.

The included audit-receiver Helm example exposes a NodePort over plain HTTP without receiver authentication. It is intentionally lab-only and should not be deployed to shared or production clusters as-is.
