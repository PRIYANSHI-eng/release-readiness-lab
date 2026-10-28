# Configuration Verification

**Candidate:** `v2.4`  
**Evidence date:** 2026-10-28

## Configuration inventory

| Variable | Application behavior | Manifest state | Production assessment |
|---|---|---|---|
| `PORT` | Defaults to `5000`; used by the development server | ConfigMap sets `"5000"` | Verified for the current container port |
| `LOG_LEVEL` | Defaults to `INFO`; normalized to uppercase | ConfigMap sets `"info"` | Verified; suitable baseline, subject to log-volume review |
| `DATABASE_URL` | Missing value selects volatile in-memory SQLite fallback | Not in ConfigMap or a Secret reference | **Blocking**; must be supplied from a managed Secret |
| `REDIS_URL` | Missing value selects a process-local in-memory cache fallback | Not in ConfigMap or a Secret reference | **Blocking**; must be supplied from a managed Secret |

## Findings

1. `k8s/configmap.yaml` correctly contains non-secret settings but explicitly omits the two external-service variables.
2. No `Secret`, `secretKeyRef`, or external secret integration is present in `k8s/`.
3. `/health` returns HTTP 200 and `"status": "healthy"` even when both required services are absent. Kubernetes readiness and liveness probes therefore cannot distinguish a production-safe pod from a fallback-only pod.
4. The application logs the configured/fallback state, but no log shipping or alerting configuration is part of this candidate.
5. No real service credentials were available for verification; none were invented or committed.

## Required configuration gate

Before approval, the deployment must receive `DATABASE_URL` and `REDIS_URL` through the target platform's secret manager, for example:

```yaml
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: checkout-api-secrets
        key: database-url
  - name: REDIS_URL
    valueFrom:
      secretKeyRef:
        name: checkout-api-secrets
        key: redis-url
```

The actual Secret must be created out-of-band with approved values. Secret values must not be placed in this repository. A staging deployment must prove that the rendered pod receives both variables and that the health contract fails or alerts when either dependency is unavailable.
