# Release Readiness Report

**Candidate:** Checkout API `v2.4`  
**Assessment date:** 2026-10-28  
**Recommendation:** **NO-GO**

## Executive decision

Do not deploy `v2.4` to production. The decision is gated by unresolved **High** risks, not by the passing unit tests alone.

## Evidence considered

- Python syntax and the two existing unit tests pass; see [validation-results.md](./validation-results.md).
- The health endpoint returns HTTP 200 with both external services absent, while reporting volatile fallback states. This makes the current Kubernetes probes insufficient evidence of production health.
- `DATABASE_URL` and `REDIS_URL` are absent from the manifests and there is no Secret integration; see [config-verification.md](./config-verification.md).
- `pip-audit` found advisories in the pinned Flask 3.0.2 and pytest 8.0.2 versions; see [dependency-checks.md](./dependency-checks.md).
- Docker image build and Kubernetes rollout validation were not completed because the local Docker engine and Kubernetes API server were unavailable.
- A concrete rollback to documented stable `v2.3` exists, but it has not yet been executed in staging; see [rollback-plan.md](./rollback-plan.md).
- Ownership roles and escalation are defined, but named window coverage and approval are still required; see [ownership.md](./ownership.md).

## Conditions to change to Go

The Release Owner may change this decision only after all of the following are attached to the change record:

1. A clean `pip-audit` result after upgrading affected dependencies and a reviewed transitive lock.
2. Secret-manager wiring for both external services, with a staging test proving real connectivity and no fallback-only healthy state.
3. Successful CI image build and security scan with an immutable image digest.
4. Successful Kubernetes staging rollout with three ready replicas, service endpoints, health/smoke results, and logs.
5. A staging rollback to `v2.3` with recorded rollout and post-rollback verification.
6. Named role owners, on-call acknowledgement, change approval, and a monitored deployment window.

Until those gates are met, approval would accept known security, availability, and recovery risks without evidence. The appropriate release action is **No-Go**.
