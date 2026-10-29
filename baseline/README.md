# Baseline Artifacts

This directory holds only this README. The original baseline evidence exists on my local machine and is excluded from the repository via `.gitignore`.

## Why it isn't published

The raw artifacts contain machine-local paths and usernames, cluster, node and workload identifiers, internal addresses, and security-posture findings. They are not safe to publish as captured.

## Local inventory (not in this repository)

- Falco 24-hour baseline text log
- kube-bench baseline screenshots
- Kubescape general-scan and NSA-framework screenshots
- Trivy baseline report screenshots

## Publication policy

A sanitized baseline summary is not currently published. If baseline results are later brought into scope, only a separately reviewed, sanitized summary will be shared, never the raw files. Before publishing anything from this set, I check it for credentials, tokens, internal addresses, usernames, cluster details, and other sensitive data.

## Limitation

The existence of a baseline log does not show that Falco telemetry stayed healthy for the full 24-hour collection period. That was not independently verified, so baseline results should not be read as proof of complete coverage.
