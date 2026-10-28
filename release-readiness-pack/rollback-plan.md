# Rollback Plan

**Candidate:** `v2.4`  
**Known-good target:** `v2.3` (documented as **Deployed (Prod - STABLE)** in `docs/architecture.md`)  
**Owner:** Release Owner with SRE/Operational Lead

## Preconditions

- Confirm the incident, timestamp, affected namespace, and current deployment image.
- Page the On-Call Support role and record the incident/approval in the deployment ticket.
- Confirm that the `v2.3` image is available in the registry by digest. Do not rely on `:latest`.
- Preserve the `v2.4` pod logs and deployment description before changing anything.

## Tested procedure

The procedure was syntax-reviewed against the repository's deployment name, namespace, and labels. It could not be executed locally because no Kubernetes API server is configured; execution in staging is a release gate.

```powershell
kubectl -n checkout-system rollout status deployment/checkout-api --timeout=120s
kubectl -n checkout-system describe deployment/checkout-api
kubectl -n checkout-system logs deployment/checkout-api --all-containers=true --since=30m

kubectl -n checkout-system set image deployment/checkout-api `
  checkout-api=kalvium/checkout-api:v2.3
kubectl -n checkout-system annotate deployment/checkout-api `
  readiness.rollback/reason="<incident-or-ticket>" --overwrite
kubectl -n checkout-system rollout status deployment/checkout-api --timeout=180s
kubectl -n checkout-system rollout history deployment/checkout-api
```

If the tagged image is mutable or unavailable, use the verified `v2.3` image digest instead:

```powershell
kubectl -n checkout-system set image deployment/checkout-api `
  checkout-api=kalvium/checkout-api@sha256:<verified-v2.3-digest>
```

## Verification and abort criteria

1. Confirm three ready replicas: `kubectl -n checkout-system get pods -l app=checkout-api`.
2. Confirm the Service has endpoints: `kubectl -n checkout-system get endpoints checkout-api-service`.
3. Run an in-cluster request to `/health` and the smoke request to `/`.
4. Confirm no new error spike in application logs and check database/cache dependency alerts.
5. Abort and escalate if rollout stalls, readiness fails, endpoints are empty, or dependency checks remain degraded.

Rollback is complete only after the SRE/Operational Lead records the rollout status, smoke results, image digest, and incident reference. A rollback does not resolve the open dependency and secret/configuration findings; those remain release blockers for `v2.4`.
