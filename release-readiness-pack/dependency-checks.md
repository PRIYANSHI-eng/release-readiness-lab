# Dependency Checks

**Evidence date:** 2026-10-28

## Pinning and runtime comparison

`app/requirements.txt` uses exact direct pins:

```text
Flask==3.0.2
gunicorn==22.0.0
pytest==8.0.2
```

These match the Flask and Gunicorn versions documented in `docs/architecture.md`, and the Dockerfile uses Python `3.11-slim`, matching the documented Python 3.11 requirement. However, transitive dependencies are not lock-pinned, so a rebuild can resolve different versions over time.

## Audit evidence

Command:

```text
pip-audit --requirement app\requirements.txt
```

Result on 2026-10-28:

```text
Found 4 known vulnerabilities in 2 packages
Name   Version ID              Fix Versions
flask  3.0.2   PYSEC-2026-2151 3.1.3
pytest 8.0.2   PYSEC-2026-1845 9.0.3
```

The duplicate rows are the auditor's repeated advisory output; the affected packages are Flask and pytest. The audit exited with status 1, so this is a release blocker. `pytest` is a test-only dependency, but it is still installed by the CI workflow and must be remediated or explicitly excluded from the production image/build policy.

## Required remediation

1. Upgrade Flask to at least `3.1.3` and pytest to at least `9.0.3` after compatibility testing.
2. Update `docs/architecture.md` and the release notes if the supported Flask version changes.
3. Generate and review a clean-environment lock file (including transitive dependencies) and make CI install from that lock.
4. Re-run `pip-audit` and require zero findings before approval.

Until that work is complete, the current exact pins are reproducible direct inputs but are not an acceptable vulnerability-free production dependency set.
