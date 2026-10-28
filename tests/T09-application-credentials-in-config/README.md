# T09-application-credentials-in-config

## Objective
Evaluate how effectively Falco's default runtime rules detect the exposure and access of application credentials embedded in Kubernetes ConfigMaps or environment variables within a running container.

## Preconditions
- Target cluster: `falco-lab` (`kind-falco-lab`).
- Falco syscall engine (`falco`) and K8s audit receiver (`falco-audit`) are healthy.
- Use only fake credential values (e.g., `fake-test-secret-123`, `dummy-api-key`). No real credentials are to be used at any point.
- Record the test start timestamp before creating the ConfigMap.

## Procedure
1. Record UTC test start time.
2. Create a Kubernetes ConfigMap containing fake credential fields:
   ```bash
   kubectl create configmap t09-fake-creds \
     --from-literal=db_password=fake-test-secret-123 \
     --from-literal=api_key=dummy-api-key-abc \
     -n falco-research
   ```
3. Deploy a pod that mounts the ConfigMap directory at `/etc/app/` and also exposes one field as an environment variable. Kubernetes projects each ConfigMap key as a file, so the password key is available at `/etc/app/db_password`:
   ```yaml
   volumes:
     - name: creds-vol
       configMap:
         name: t09-fake-creds
   containers:
     - name: t09-creds-test
       image: busybox:1.38.0
       command: ["sleep", "3600"]
       env:
         - name: DB_PASSWORD
           valueFrom:
             configMapKeyRef:
               name: t09-fake-creds
               key: db_password
       volumeMounts:
         - name: creds-vol
           mountPath: /etc/app
   ```
4. Monitor K8s audit logs for pod creation alerts within the 60-second window. Note any alerts on ConfigMap access.
5. Exec into the pod and access the credential file:
   ```bash
   kubectl exec -it t09-creds-test -n falco-research -- cat /etc/app/db_password
   ```
6. Also print the environment variable:
   ```bash
   kubectl exec -it t09-creds-test -n falco-research -- sh -c 'echo $DB_PASSWORD'
   ```
7. Monitor Falco syscall logs for the 60-second detection window from the exec timestamp. Look specifically for:
   - `Read sensitive file untrusted` (if `/etc/app/` path matches the sensitive path macro)
   - `Read environment variable from /proc files` or similar env-scraping rules
   - `Attach/Exec to pod` (k8saudit rule triggered on kubectl exec)
8. Record all observed alerts, rule names, priorities, and calculate latency.
9. Delete the test pod and the ConfigMap after the detection window closes.

## Success criteria
- Technique execution: The container reads the mounted credential file and the environment variable successfully.
- Detection: Falco emits at least one alert identifying the credential file read or the exec session within 60 seconds.
- Cleanup: The test pod and ConfigMap are fully removed.

## Evidence
- Evidence log: `evidence/T09/T09-application-credentials-in-config-2026-10-28.txt`
  - Pre-test state: pod and ConfigMap list, Falco health check
  - Test execution: kubectl create, apply, and exec outputs
  - Detection window: Falco log extract (60 seconds from exec timestamp)
  - Cleanup confirmation: delete outputs

## Outcome
- Test status: Completed (28 October 2026)
- Technique execution: Succeeded. Fake values were read from the mounted ConfigMap file and environment variable.
- Telemetry availability: Syscall and Kubernetes audit streams available.
- Observed alert: `Attach/Exec to pod` (Notice) at approximately 0.34 seconds; no credential-specific syscall alert observed in the 60-second window.
- Classification: **Context Only**. The exec-session alert identifies the test mechanism, not the credential read itself.
- Cleanup: Confirmed. Temporary pod and ConfigMap removed.
- Evidence: `evidence/T09/T09-application-credentials-in-config-2026-10-28.txt` (local archive).
