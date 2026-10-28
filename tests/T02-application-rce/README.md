# T02-application-rce

## Objective
Evaluate how effectively Falco's default rules detect remote code execution (RCE) / unauthorized process execution inside a Kubernetes application container.

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
- Detection: Falco emits a specific or generic alert attributing the execution within 60 seconds.

## Evidence
- Evidence log (local archive): `evidence/T02/T02-application-rce-2026-10-22.txt`

## Outcome
- Test status: Completed
- Technique execution: Succeeded (command executed as `root` in target container).
- Default alert: `Attach/Exec to pod` (Priority Notice) via K8s audit; complementary `Terminal shell in container` on interactive TTY via syscall.
- Alert latency: ~1.29 seconds.
- Classification: **Specific** (Detected by default rules).
- Retest: Not required (detected by default configuration).
