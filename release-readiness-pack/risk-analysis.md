# Risk Analysis

**Assessment date:** 2026-10-28  
**Decision basis:** highest open risk governs the release decision.

| Risk | Severity | Likelihood | Mitigation / acceptance condition | Owner |
|---|---|---|---|---|
| Missing `DATABASE_URL` and `REDIS_URL` causes volatile, non-distributed fallbacks | High | High | Provision both through the target secret manager; test real staging connectivity; change health/readiness behavior so fallback is not healthy | Application Owner + SRE |
| Flask 3.0.2 and pytest 8.0.2 have known advisories from `pip-audit` | High | High | Upgrade to fixed versions, compatibility-test, regenerate a transitive lock, and obtain a clean audit | Application Owner |
| Image build was not executed because Docker daemon was unavailable; registry image provenance/digest is unverified | High | Medium | Build in CI, publish an immutable digest, scan it, and record the digest in the deployment evidence | Release Owner |
| Kubernetes rollout and rollback were not exercised against a cluster | High | Medium | Run in staging, capture rollout/readiness/service evidence, then perform a controlled rollback to `v2.3` | SRE/Operational Lead |
| `/health` returns HTTP 200 while dependencies are absent | High | High | Define dependency-aware readiness semantics and add a negative test; do not approve current probes as production health | Application Owner |
| No committed Secret integration or external-secret contract | High | High | Add reviewed `secretKeyRef`/external-secret wiring and verify values out-of-band without committing credentials | SRE/Operational Lead |
| Three replicas share no documented disruption/PDB or capacity test | Medium | Medium | Run a load/disruption test; add a PodDisruptionBudget and autoscaling/capacity decision if required by SLO | SRE/Operational Lead |
| Deployment approval and named deploy-window coverage are absent | Medium | Medium | Record approver, change ticket, deploy window, and on-call acknowledgement before Go | Release Owner |

## Risk disposition

The High risks are unresolved. They are not acceptable as assumed operational debt for a production deployment. The release remains **No-Go** until the configuration, dependency, image, and staging rollout gates have evidence.
