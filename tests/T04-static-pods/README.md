# T04-static-pods

## Objective
Evaluate default Falco detection of unauthorized Kubernetes static-pod creation on control-plane nodes, and validate the custom manifest-write detection rule.

## Preconditions
- Target: the dedicated kind control-plane node for `falco-lab`.
- Falco syscall engine is healthy with the modern eBPF driver.
- Custom rule `t04-static-pod.yaml` mounted under `/etc/falco/rules.d/`.
- Ensure original control-plane manifests (`etcd.yaml`, `kube-apiserver.yaml`, `kube-controller-manager.yaml`, `kube-scheduler.yaml`) are backed up.

## Procedure
1. Verify pre-test state of `/etc/kubernetes/manifests/` on the kind control-plane node.
2. Record UTC test start time.
3. Write a temporary static-pod manifest `/etc/kubernetes/manifests/t04-staticpod-verify.yaml` using `busybox:1.38.0`. Pull it into the kind node first if it is not cached.
4. Monitor Falco logs for default rules and custom rule alerts within the 60-second window.
5. Capture exact alert details: rule name, process, user, file path, event type, and latency.
6. Remove the temporary static-pod manifest immediately.
7. Verify all four core control-plane manifests remain unchanged.

## Success criteria
- Technique execution: Manifest is written to `/etc/kubernetes/manifests/`.
- Default detection: Evaluate default Falco rules only after confirming manifest-write telemetry; otherwise classify the baseline as inconclusive.
- Custom detection: Custom rule emits `T04_STATIC_POD_MANIFEST_WRITE` with Warning priority.
- Cleanup: Temporary manifest removed; node control-plane manifests preserved.

## Evidence
- Baseline log: `evidence/T04/T04-static-pods-2026-10-22.txt`
- Custom retest log: `evidence/T04/T04-custom-rule-static-pod-retest-2026-10-22.txt`

## Outcome
- Original test: The static pod ran, but syscall telemetry for the manifest write was not independently validated at that time.
- Validated retest (28 October 2026): Falco was Ready with modern eBPF and the T04 rule loaded. Writing a harmless pause static-pod manifest produced the expected `T04_STATIC_POD_MANIFEST_WRITE` syscall event during the 60-second window. No relevant default rule alert was observed.
- Default-rule classification: **No Detection** for this tested manifest-write path. This is not a claim about every static-pod creation path.
- Custom-rule result: **Detected** on the same actual manifest write.
- Cleanup: Confirmed. Temporary manifest and static pod were removed; SHA-256 hashes for all four original control-plane manifests were unchanged.
- Evidence: `evidence/T04/T04-default-telemetry-retest-2026-10-28.txt` (local archive).
