# Benchmarks

Numbers measured against the reference course laptop. Record your own
measurements in the same table — your machine will differ, and that is
expected; what matters is the *shape* of the result (which row is faster,
and by roughly how much), not matching these numbers exactly.

## Day 1 — Module 2 (FastAPI hardening)

| Metric | `async def` (bug) | plain `def` (fixed) |
|---|---|---|
| p50 latency | — | 15 ms |
| p99 latency | 2.4 s | 38 ms |
| Throughput (c=25) | 118 req/s | 1,410 req/s |

- Load test: `hey -n 2000 -c 25 -m POST -D sample.json`
- Malformed corpus: 40/40 payloads rejected with 4xx (`payloads/malformed/`)
- Valid-traffic error rate: 0 non-2xx responses

## Day 2 — Lab 3 (Docker) — done

| Metric | Value |
|---|---|
| Naive build — image size | 2.41 GB |
| Naive build — cold build time | 6 min 12 s |
| Multi-stage build — image size | 412 MB (target ≤ 450 MB) |
| Multi-stage build — cold build time | ~1 min (varies by machine) |
| Warm rebuild (one-line code change) | 22 s, `CACHED` on the dependency layer |
| p99 — bare metal (Lab 2, no container) | 38 ms |
| p99 — containerised | 41 ms |
| Time-to-ready (`scripts/startup_time.sh`) | 6.8 s (target: under 10 s) |

Image ships as a non-root `appuser`, healthcheck on `/v1/ready`,
`docker compose ps` shows both `fraud-api` and `feature-cache` as
`(healthy)`.

## Day 2 — Lab 4 (Tests) — done

| Metric | Value |
|---|---|
| `pytest -m "not slow"` — pass count / duration | 52 passed in ~2.1 s |
| `pytest -m slow` — pass count / duration | 3 passed in ~5.9 s (includes the 5,000-row golden-file check) |
| Branch coverage (domain + service + api) | ~99% (target ≥ 80%) |
| Malformed corpus | 40/40 rejected with 4xx |

## Day 3 — Lab 5 (CI/CD pipeline)

### Gate commands, measured locally (what each CI job actually runs)

| Metric | Value |
|---|---|
| `ruff check src tests` | 0.9 s cold / 0.1 s warm |
| `lint-imports` (architecture contract) | 0.3 s — Contracts: 1 kept, 0 broken |
| `mypy src/fraud_service --strict` | Success: no issues in 15 source files |
| `pytest -m "not slow" --cov-fail-under=80` | 52 passed in 2.5 s, branch coverage 98.66% |
| `pytest -m behavioural --no-cov` | 3 passed in 6.1 s (real model artefact) |

### GitHub Actions run (fill in from the Actions UI after the first push)

| Metric | Value |
|---|---|
| lint job duration | |
| test job duration | |
| image-smoke — cold run | _(course reference: ~5 min 40 s)_ |
| image-smoke — warm run (GHA cache) | _(course reference: ~1 min 02 s)_ |
| publish job duration | |
| bad-pr blocked by branch protection? | yes / no |

### bad-pr — both gates proven red before the fix

| Check | Result on `bad-pr` |
|---|---|
| `ruff check` | passes — style linting does not catch either bug, which is the point |
| `lint-imports` | **BROKEN** — `fraud_service.domain.policies -> fraud_service.api.schemas (l.6)` |
| `pytest -m "not slow"` | **1 failed**, 51 passed — `test_decision_bands[0.85-block]`: `assert 'review' == 'block'` |
| after the proper fix (import removed, `>=` restored) | all four gates green again, no test weakened |

**Note on the test job.** The lab guide prints the behavioural step as
`pytest -m "behavioural and not slow"`. In this repository
`tests/behavioural/` sets `pytestmark = [behavioural, slow]`, so that selector
collects **zero** tests and pytest exits with code 5 — a red job that ran no
tests at all. The step therefore runs `pytest -m behavioural --no-cov`:
`--no-cov` because `fail_under = 80` in `pyproject.toml` is global, and a
behavioural-only run reports ~65% coverage, failing a gate that the fast-suite
step already enforces on the full suite.

## Day 3 — Lab 6 (Config, Secrets & Logs)

_Fill in after Lab 6 Step 3:_

| Metric | Value |
|---|---|
| p50 latency computed from JSON logs via `jq` | |
| Fail-fast startup error (bad `FRAUD_MODEL_PATH`) confirmed? | yes / no |
| `gitleaks` clean on final commit? | yes / no |
