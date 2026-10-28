# GitHub Safe Release Package

This public repository contains the reviewed project sources assembled from an explicit file allowlist. The original local Git history and all raw test evidence, baseline screenshots/logs, backups, kubeconfig, credentials, and private keys were excluded. The repository contains the sanitized evidence register and current result tracker.

## Assessment status

All twelve scenarios have a recorded, classifiable outcome: three specific detections, four generic detections, and five no-detection results. The observed relevant-alert rate is 7/12 (58.3%). T08's generic exec alert does not identify proxy use or the outbound connection. T12 tested three named in-cluster services/ports, not a comprehensive scan. This is a bounded lab assessment, not a comprehensive security certification.

Supplemental September 2026 kube-bench, Kubescape, and Trivy baseline captures were reviewed for publication sensitivity but are not incorporated into the Falco analysis; their currentness and completeness are not established. Raw baseline material is deliberately absent.

The current T03 rule source is broader than the version exercised in its retest because it removes a Kyverno identity exclusion. Validate that expanded rule before deployment.

## Publication safety

1. Do not copy local `baseline/`, `evidence/`, or `backups/` contents into this repository.
2. Review and rescan any new files before committing them.
3. No license is included. Add one only after choosing the reuse terms you want.

This package was assembled from an explicit file allowlist. It was scanned for common credential/private-key patterns; that check is a precaution and does not prove absence of every sensitive value.

The included audit-receiver Helm example exposes a NodePort over plain HTTP without receiver authentication. It is intentionally lab-only and should not be deployed to shared or production clusters as-is.

