# Kubernetes Detection Coverage Gap Analysis

This project measures how Falco detects a defined set of Kubernetes and container security scenarios, documents verified gaps, and validates targeted custom rules. Findings follow **verify, don't assume**: test success, telemetry health, observed alerts, and classification are tracked separately.

> **Lab-only:** Several procedures create cluster-admin bindings, hostPath mounts, host-network pods, or static-pod manifests. Run them only in a disposable, isolated lab after reviewing each procedure. Do not apply the test steps to production clusters.

## Current assessment status

All twelve planned scenarios have recorded, classifiable outcomes: three specific detections (T02, T05, T06), four generic detections (T08, T09, T11, T12), and five no-detection results (T01, T03, T04, T07, T10). Custom-rule retests confirmed T01, T03, T04, and T07 behavior. No overall post-custom coverage rate is claimed.

The current observed relevant-alert rate is **7/12 (58.3%) among classifiable scenarios**. This is a bounded result from the recorded lab configuration, not a general product effectiveness estimate. T08's generic exec alert does not identify the proxy connection; T12 tested three named in-cluster services/ports, not a comprehensive scan.

## Project contents

- [`docs/`](docs/README.md) — lab setup, methodology, and classification criteria
- [`tests/`](tests/README.md) — twelve scenario procedures and status index
- [`rules/custom/`](rules/README.md) — custom Falco rules and implementation notes
- [`helm/`](helm/README.md) — runtime and audit receiver configuration
- [`results/detection-coverage.csv`](results/detection-coverage.csv) — machine-readable tracker
- [`reports/`](reports/README.md) — executive summary, technical report, sanitized evidence register
- `evidence/`, `backups/`, and `baseline/` — local-only evidence/archive material; excluded from Git

## Assessment workflow

1. Review [lab setup](docs/lab-setup.md) and [methodology](docs/methodology.md).
2. Confirm telemetry for the source relevant to the test (syscall or Kubernetes audit).
3. Execute one isolated scenario from `tests/` and record the UTC action timestamp.
4. Preserve raw output locally; publish only sanitized excerpts.
5. Classify using [detection criteria](docs/detection-criteria.md), update the CSV, and reconcile the test record and reports.
6. Retest custom rules separately from the default baseline.

## Publication and evidence

Raw evidence, backups, and baseline captures are excluded from this repository because they contain environment identifiers or sensitive security details. The checked-in evidence register contains sanitized summaries; do not copy local raw captures into the public repository.

