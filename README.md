# Kubernetes Detection Coverage Gap Analysis

This repository contains my Falco detection coverage assessment for a small Kubernetes lab. It records twelve test scenarios, what Falco alerted on, and a few custom rules written to cover gaps found during testing. The results are tied to the setup described here; they are not a general score for Falco.

> **Lab-only:** Several procedures create cluster-admin bindings, hostPath mounts, host-network pods, or static-pod manifests. Run them only in a disposable, isolated lab after reviewing each procedure. Do not apply the test steps to production clusters.

## Current assessment status

I ran all twelve scenarios and was able to classify each result: three specific detections (T02, T05, T06), four generic detections (T08, T09, T11, T12), and five cases with no relevant default alert (T01, T03, T04, T07, T10). I also retested custom rules for T01, T03, T04, and T07. Those retests covered only part of the suite, so I have not calculated a post-custom-rule coverage rate.

Across these runs, Falco produced a relevant alert in **7 of 12 scenarios (58.3%)**. T08's alert shows the `kubectl exec` session, not the proxy connection. T12 checked three named in-cluster services and ports; it was not a broad network scan.

## Project contents

- [`docs/`](docs/README.md) — lab setup, methodology, and classification criteria
- [`tests/`](tests/README.md) — twelve scenario procedures and status index
- [`rules/custom/`](rules/README.md) — custom Falco rules and implementation notes
- [`helm/`](helm/README.md) — runtime and audit receiver configuration
- [`results/detection-coverage.csv`](results/detection-coverage.csv) — machine-readable tracker
- [`reports/`](reports/README.md) — executive summary, technical report, sanitized evidence register
- `evidence/`, `backups/`, and `baseline/` — placeholder README files are tracked; raw evidence and archives are kept locally and excluded from Git

## Assessment workflow

1. Review [lab setup](docs/lab-setup.md) and [methodology](docs/methodology.md).
2. Confirm telemetry for the source relevant to the test (syscall or Kubernetes audit).
3. Execute one isolated scenario from `tests/` and record the UTC action timestamp.
4. Preserve raw output locally; publish only sanitized excerpts.
5. Classify using [detection criteria](docs/detection-criteria.md), update the CSV, and reconcile the test record and reports.
6. Retest custom rules separately from the default baseline.

## Publication and evidence

I have kept raw evidence, backups, and baseline captures out of this repository because they include environment details. The evidence register contains summaries with identifying values removed. Keep the original captures in the local archive; do not add them to the public repository.
