# Deployment Ownership

This ownership model must be filled with named people in the change ticket and PR approval before deployment. Role ownership is assigned below so the deployment cannot proceed as an orphaned change.

| Role | Accountable responsibility | Required hand-off / evidence |
|---|---|---|
| Release Owner | Release Engineering / repository owner (`PRIYANSHI-eng`) | Owns the go/no-go decision, change record, PR approval, deployment window, and final evidence index |
| Application Owner | Checkout API development team | Owns Flask behavior, dependency remediation, health semantics, and application smoke tests |
| SRE / Operational Lead | Production SRE team | Owns secrets integration, Kubernetes rollout, observability, capacity, and rollback execution |
| On-Call Support | Production on-call engineer named in the change ticket | Acknowledges the window, watches alerts/logs, runs smoke checks, and opens/escalates the incident |
| Escalation Path | SRE lead -> Application Owner -> Release Owner -> service/platform leadership | Escalate immediately for failed readiness, dependency outage, security finding, or rollback failure |

## Minimum approval record

Before the deployment window, the change record must contain:

- named Release Owner, Application Owner, SRE/Operational Lead, and On-Call Support;
- start/end time and communication channel;
- approved change/incident reference;
- evidence links for clean dependency audit, image digest scan, staging rollout, and tested rollback;
- explicit approver acknowledgement of the readiness report.

No production credentials belong in this repository or in the PR.
