# T02-application-rce

## Objective
Record the alerts generated when a shell command is run in an application container through `kubectl exec`. This is an RCE-like command-execution simulation; it does not exploit an application vulnerability and should not be described as a test of exploit detection.

## Preconditions
- Target: `falco-lab` (`falco-research` namespace).
- Target pod: `sidecar-target` deployment.
- Telemetry engines: Modern eBPF syscall engine (`falco`) and K8s audit receiver (`falco-audit`).

## Procedure
1. Verify target container status.
2. Record UTC test start time.
3. Execute remote command execution payload into the container (`sh -c id; hostname; uname -a`).
4. Monitor Falco audit and syscall logs for detection alerts within 60 seconds.
5. Capture alert priority, rule name, timestamp, and calculate latency.

## Success criteria
- Technique execution: Command executes inside target container and returns expected output.
- Detection: Falco emits an alert that identifies the container command execution. An alert for the `kubectl exec` API request alone identifies the test mechanism, not RCE.

## Evidence
- Evidence log (local archive): `evidence/T02/T02-application-rce-2026-10-22.txt`

## Outcome
- Test status: Completed
- Technique execution: Succeeded (command executed as `root` in target container).
- Default alert: `Attach/Exec to pod` (Priority Notice) via Kubernetes audit. The evidence also notes that `Terminal shell in container` is available for interactive TTY use, but does not record it as an alert from this test run.
- Alert latency: ~1.29 seconds.
- Classification: **Context Only**. The observed alert identifies the `kubectl exec` mechanism, not an application exploit or the command's behavior.
- Retest: Not required (detected by default configuration).
