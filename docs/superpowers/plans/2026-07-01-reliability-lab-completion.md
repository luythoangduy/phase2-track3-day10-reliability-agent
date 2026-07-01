# Reliability Lab Completion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete the reliability lab so tests pass, chaos metrics are generated, the final report is reproducible, and the branch can be pushed.

**Architecture:** The gateway checks cache first, then calls providers through per-provider circuit breakers, then returns a static fallback if all providers fail. Metrics are collected during chaos scenarios and exported as JSON/CSV/report artifacts.

**Tech Stack:** Python 3.10+, pytest, pydantic, Redis via Docker Compose, redis-py.

---

### Task 1: Circuit Breaker

**Files:**
- Modify: `src/reliability_lab/circuit_breaker.py`
- Test: `tests/test_circuit_breaker.py`

- [ ] Run `pytest tests/test_circuit_breaker.py -q` to confirm the current failures.
- [ ] Implement `allow_request()` with CLOSED, OPEN timeout, and HALF_OPEN probe behavior.
- [ ] Implement `call()` so successful calls record success and exceptions record failure before re-raising.
- [ ] Implement `record_success()` and `record_failure()` with separate HALF_OPEN transition reasons.
- [ ] Run `pytest tests/test_circuit_breaker.py -q` and require all tests to pass.

### Task 2: Cache Layers

**Files:**
- Modify: `src/reliability_lab/cache.py`
- Test: `tests/test_cache.py`, `tests/test_redis_cache.py`

- [ ] Run `pytest tests/test_cache.py -q` to confirm the current failures.
- [ ] Implement in-memory `ResponseCache` TTL eviction, privacy bypass, false-hit logging, and n-gram cosine similarity.
- [ ] Implement `SharedRedisCache.get()` and `SharedRedisCache.set()` using Redis hashes with TTL and similarity scan fallback.
- [ ] Start Redis with `docker compose up -d`.
- [ ] Run `pytest tests/test_cache.py tests/test_redis_cache.py -q` and require all tests to pass with Redis running.

### Task 3: Gateway Pipeline

**Files:**
- Modify: `src/reliability_lab/gateway.py`
- Test: `tests/test_gateway_contract.py`

- [ ] Run `pytest tests/test_gateway_contract.py -q` to confirm failures.
- [ ] Implement `ReliabilityGateway.complete()` with cache hit, provider fallback, circuit open skip, cache population, and static fallback response.
- [ ] Run `pytest tests/test_gateway_contract.py -q` and require all tests to pass.

### Task 4: Chaos And Metrics

**Files:**
- Modify: `src/reliability_lab/chaos.py`
- Modify: `src/reliability_lab/metrics.py`
- Test: `tests/test_metrics.py`, `tests/test_todo_requirements.py`

- [ ] Implement `calculate_recovery_time_ms()` from breaker transition logs.
- [ ] Implement `run_scenario()` to collect request counts, successes, fallbacks, cache hits, circuit opens, cost, and latencies.
- [ ] Implement `RunMetrics.write_csv()` with flattened `scenario_*` columns.
- [ ] Run `pytest tests/test_metrics.py tests/test_todo_requirements.py -q` and require the TODO xfail tests to XPASS.

### Task 5: Reports And Verification

**Files:**
- Generate: `reports/metrics.json`
- Generate: `reports/final_report.md`
- Optional generate: `reports/metrics.csv`

- [ ] Run the full suite with `pytest -q` while Redis is running.
- [ ] Run `python scripts/run_chaos.py --config configs/default.yaml --out reports/metrics.json`.
- [ ] Run `python scripts/generate_report.py --metrics reports/metrics.json --out reports/final_report.md`.
- [ ] Replace the generated report analysis section with concrete architecture, configuration, metrics, Redis evidence, chaos observations, failure analysis, and next steps.
- [ ] Run `git status --short`, commit the scoped changes, and push `main` to `origin`.

