# Detection Coverage Results

[`detection-coverage.csv`](detection-coverage.csv) is the machine-readable record for the twelve planned scenarios. It separates execution success, telemetry availability, observed default alert, default classification, custom rule, and retest result.

## Classification values

- `Specific` — alert identifies the behavior under test.
- `Generic` — alert identifies related activity but not the behavior under test precisely.
- `Context Only` — alert identifies the test harness or setup; it is not counted as detection.
- `No Detection` — successful test, required telemetry available, and no relevant alert within 60 seconds.
- `Inconclusive` — evidence does not establish relevant telemetry delivery or that the matching rule was loaded; not counted as a miss.
- `Not Run` — no test outcome.

Custom retest status is recorded independently. A custom alert does not rewrite the default result. Evidence paths refer to the local, Git-ignored evidence archive; the sanitized summaries available to repository readers are in [`reports/evidence-register.md`](../reports/evidence-register.md).

Before changing results, preserve the previous tracker in the local `backups/results/` archive, update the relevant test record and evidence, then reconcile the reports and computed denominator. Never count inconclusive or unrun scenarios as misses.
