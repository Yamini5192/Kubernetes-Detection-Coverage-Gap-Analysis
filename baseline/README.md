# Baseline Artifacts

The original baseline evidence is stored locally in this directory and is excluded from Git by default.

The raw Falco log and scanner screenshots contain machine-local paths and usernames, cluster/node/workload identifiers, internal addresses, and security-posture findings. Do not publish these raw artifacts. Only share a separately reviewed, sanitized summary if the baseline results are intentionally in scope for release.

Current inventory:
- Falco 24-hour baseline text log
- kube-bench baseline screenshots
- Kubescape general-scan and NSA-framework screenshots
- Trivy baseline report screenshots

Before publishing any artifact, inspect it for credentials, tokens, internal addresses, usernames, cluster details, and other sensitive data. Publish only reviewed, sanitized copies. The existence of a baseline log alone does not establish that telemetry was healthy throughout the full collection period.
