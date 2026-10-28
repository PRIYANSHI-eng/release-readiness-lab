# Validation Results

**Candidate:** Checkout API `v2.4`  
**Evidence date:** 2026-10-28  
**Branch:** `feature/readiness-pack`

## Evidence summary

| Check | Command / source | Result |
|---|---|---|
| Python syntax | `python -m py_compile app\app.py app\test_app.py` | Passed |
| Unit tests | `pytest app\test_app.py -q` | **2 passed** in 0.63s |
| Dependency installation | `python -m pip install -r app\requirements.txt` | Passed; exact direct pins installed |
| Health endpoint without external services | Flask test client, no `DATABASE_URL` or `REDIS_URL` | HTTP 200, but both checks reported fallback |
| Health endpoint with configuration | Flask test client with redacted placeholder URLs | HTTP 200, both checks reported `connected` |
| Container build | `docker build --file app\Dockerfile --tag checkout-api:v2.4 .\app` | **Not run: Docker Desktop Linux engine unavailable** |
| Kubernetes validation | `kubectl apply --dry-run=client ...` | **Not run: no Kubernetes API server configured** |
| CI evidence | `.github\workflows\ci.yml` | Workflow covers syntax, tests, and a dry-run image build; no completed run is attached to this branch |

## Reproducible local output

```text
..                                                                       [100%]
2 passed in 0.63s
```

The fallback health check produced:

```text
200
{'checks': {'database': 'warning_fallback_sqlite',
            'redis': 'warning_fallback_local'},
 'status': 'healthy',
 'version': 'v2.4'}
```

With placeholder values supplied for both required variables it produced:

```text
200
{'checks': {'database': 'connected',
            'redis': 'connected'},
 'status': 'healthy',
 'version': 'v2.4'}
```

The placeholder values were never committed and were not used to contact a real service.

## Limitations and required follow-up

The passing unit suite does not prove connectivity to PostgreSQL or Redis because the application currently reports configured URLs as connected without opening a connection. A staging run with real, non-production services is required before approval.

The image build must be repeated with Docker available, and the manifests must be applied to a disposable Kubernetes namespace. Record the image digest, pod readiness, service reachability, and rollout result in the PR before changing the decision to Go.
