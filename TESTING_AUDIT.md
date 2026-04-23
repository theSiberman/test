# Testing & Developer Experience Audit

## Context
This repository appears to be in an early/minimal state (`README.md` only), so there is no executable test suite to profile directly yet.

To still unblock your request, this audit provides a **practical upgrade blueprint** for projects where test coverage is strong but execution time has become a bottleneck.

## Symptoms You Described
- Tests are extensive and likely high confidence.
- End-to-end runtime is now too slow for local iteration.
- Slow feedback is throttling development velocity.

---

## Recommended Testing Architecture (Fast Feedback First)

Use a **3-tier testing model** and enforce each tier at different phases:

1. **Tier 1: PR Gate (2–8 minutes target)**
   - Unit tests
   - Component/smoke integration tests
   - Lint/type checks
   - Must run on every PR and on each push to PR branch.

2. **Tier 2: Full CI (10–30+ minutes target)**
   - All integration tests
   - Contract tests
   - Cross-platform matrix as needed
   - Run on merge queue, `main` pushes, or manually.

3. **Tier 3: Nightly/On-demand (long-running)**
   - E2E suites
   - Performance/regression tests
   - Flake detection runs (repeat mode)

### Why this helps
It preserves quality while restoring fast day-to-day iteration by gating code review on confidence-critical tests rather than the entire suite.

---

## Immediate Changes to Improve Velocity

### 1) Test Impact Analysis (TIA)
Run only tests related to changed files during PR checks.
- Map files -> test groups.
- Fall back to wider runs when mapping is uncertain.
- Keep a manual label (`full-ci`) to force full suite.

### 2) Parallelization + Sharding
- Split integration/E2E into shards by historical runtime.
- Rebalance shards weekly based on timing data.
- Ensure deterministic test isolation (no shared mutable state).

### 3) Smart Caching in CI
- Cache dependency installs.
- Cache compiled artifacts where valid.
- Cache test framework state (if supported) safely.

### 4) Flake Budget & Quarantine Workflow
- Track flaky tests in a quarantine list.
- Keep quarantined tests out of PR gate but run nightly.
- Enforce an SLO: e.g. “flake rate < 1% over 7 days.”

### 5) Local Developer Presets
Provide standardized commands:
- `test:quick` (sub-5 min)
- `test:changed` (impacted tests only)
- `test:full` (all deterministic tests)
- `test:e2e` (optional local)

---

## PR / CI / Code Review Upgrade Plan

### PR Workflow
- Require PR template fields:
  - Risk level
  - Test scope executed locally
  - Whether `full-ci` is requested
  - Rollback notes for risky changes
- Auto-label PRs by touched paths (e.g., `backend`, `frontend`, `infra`).

### CI Workflow Design
Create two key workflows:

1. **Fast PR Checks**
   - Trigger: `pull_request`
   - Jobs: lint, typecheck, unit, smoke integration
   - Runtime target: <= 8 minutes

2. **Comprehensive Validation**
   - Trigger: `push` to `main`, `workflow_dispatch`, nightly schedule
   - Jobs: full integration, e2e, non-blocking reliability jobs

### Code Review Standards
- Add lightweight “test evidence” requirement in PR description.
- Require reviewers to verify:
  - test scope is adequate for risk
  - changed behavior has at least one direct test
- Use CODEOWNERS for critical areas.

---

## Suggested Metrics Dashboard
Track these weekly:
- PR gate median runtime (P50/P95)
- Time-to-first-failure in CI
- Flake rate
- Mean queue wait time
- Re-run rate by workflow
- % PRs requiring full CI override

Use these to tune thresholds and decide where to invest.

---

## 30-Day Rollout Plan

### Week 1
- Baseline current CI timings and flake rate.
- Introduce fast PR workflow only.

### Week 2
- Add changed-tests mode and caching.
- Add PR template test evidence section.

### Week 3
- Shard long suites by historical duration.
- Launch quarantine process for flaky tests.

### Week 4
- Turn on nightly full reliability run.
- Review metrics and tighten SLOs.

---

## Risks & Mitigations

- **Risk:** Missing regressions with reduced PR gate.
  - **Mitigation:** Keep nightly full runs + merge queue full CI.

- **Risk:** False confidence from flaky tests.
  - **Mitigation:** quarantine + flake SLO ownership.

- **Risk:** Developer confusion over test commands.
  - **Mitigation:** single docs page + consistent command naming.

---

## Recommended Next Implementation Tasks
1. Add PR template and CI split workflows (provided in this change set).
2. Define your test command contracts (`quick`, `changed`, `full`) in project scripts.
3. Add test timing collection and publish artifact for shard balancing.
4. Add CODEOWNERS + review checklist to institutionalize quality.
