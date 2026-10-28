# Detection Criteria

The detection window is 60 seconds from the recorded technique timestamp.

- **Specific** — an alert identifies the tested technique or a sufficiently precise behavior to attribute it to that technique.
- **Generic** — a relevant alert occurs, but it does not identify the technique with sufficient specificity. Alerts arriving more than 60 seconds after the test are also recorded as late and do not count as timely detection.
- **No Detection** — the technique succeeded, the required telemetry was available, and no relevant alert arrived within the detection window.
- **Inconclusive** — the technique did not succeed, evidence is insufficient, or the required telemetry was unavailable.

For analysis, keep the observed alert, timeliness, technique success, and telemetry availability as separate fields. Do not count an inconclusive test as a detection miss.
