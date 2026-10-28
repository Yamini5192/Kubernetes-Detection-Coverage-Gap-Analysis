# GitHub Safe Release Package

This public repository contains the project files selected for release. I left out the original local Git history, raw test evidence, baseline screenshots and logs, backups, kubeconfig, credentials, and private keys. The evidence register contains sanitized summaries, alongside the current result tracker.

## Assessment status

I attempted all twelve scenarios. Two classifiable scenarios produced specific detections, five produced only context alerts from the test mechanism or setup, four had no relevant default alert, and T10 is inconclusive because event-level audit delivery and loaded-rule status were not documented. Only **2 of 11 classifiable scenarios (18.2%)** produced an alert that identified the behavior under test. These results describe this lab assessment only.

The loaded Falco ruleset had no Secret-list rule. A metadata-only project rule was added and retested successfully, while the original default-baseline result remains inconclusive because its capture did not show list-event delivery. Supplemental September 2026 kube-bench, Kubescape, and Trivy baseline captures were reviewed for publication sensitivity but are not incorporated into the Falco analysis; their currentness and completeness are not established. Raw baseline material is deliberately absent.

The broader T03 source without a Kyverno identity exclusion was loaded and retested under the Kyverno admission-controller identity. It fired as expected; tune for legitimate controller changes before wider deployment.

## Publication safety

1. Do not copy local `baseline/`, `evidence/`, or `backups/` contents into this repository. The tracked directories contain placeholder README files only.
2. Review and rescan any new files before committing them.
3. This project is released under the MIT License in `LICENSE`.

I checked the release files for common credential and private-key patterns. That check is useful as a precaution, but it cannot guarantee that every sensitive value has been found.

The T08 and T12 procedures depend on Kubernetes Goat and `health-check-service` workloads that are installed separately and are not included here. See `docs/lab-setup.md` before running them.

The included audit-receiver Helm example exposes a NodePort over plain HTTP without receiver authentication. It is intentionally lab-only and should not be deployed to shared or production clusters as-is.
