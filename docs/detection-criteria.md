# Detection Criteria

The detection window is 60 seconds from the recorded technique timestamp.

- **Specific** — an alert identifies the behavior under test with enough detail to distinguish it from the test harness or setup.
- **Generic** — an alert identifies related activity, but not the behavior under test with enough specificity.
- **Context Only** — an alert identifies the mechanism used to run the test (for example, `kubectl exec`) or a required setup step, but gives no evidence that the behavior under test was detected. Do not count this as a detection.
- **No Detection** — the technique succeeded, the required telemetry was available, and no relevant alert arrived within the detection window.
- **Inconclusive** — the action succeeded or failed, but evidence is insufficient to determine whether the relevant telemetry reached Falco or whether the ruleset could match it.

Alerts arriving more than 60 seconds after the test are recorded as late and do not count as timely detection. An active receiver or a nearby audit event does not, by itself, prove that the event type under test was delivered.

For analysis, keep the observed alert, timeliness, technique success, and telemetry availability as separate fields. Do not count an inconclusive test as a detection miss.
