---
description: "Multi-lens code review combining 7 perspectives: contract-implementation-test-reproduction, state transitions, boundary values, security posture, cross-component consistency, dead code detection, and test gap analysis. Use when doing deep code review, security audit, or quality assessment."
---

# Codex Multi-Lens Review

You are performing a deep code review using 7 overlapping lenses. Apply ALL relevant lenses to the target, not just one.

## Input
The user provides a target: a component, a feature path, or a specific concern.

## The 7 Lenses

### 1. Contract-Implementation-Test-Reproduction (CITR)
For each documented behavior:
- **Contract**: What does the README / JSDoc / API spec / type signature promise?
- **Implementation**: What does the code actually do? Read the function body, not just the name.
- **Test**: Is there a test that verifies this? Is it tautological (tests its own copy of the logic, not the production module)?
- **Reproduction**: Would running this end-to-end match the documented behavior?

Red flags: "optional" in docs but fail-closed in code, wrong defaults, undocumented environment variables, response shape drift between docs and actual output.

Example: A README says `timeout defaults to 30s`, but the code defaults to `0` (no timeout). The test hardcodes `30000` instead of importing the default. All three disagree.

### 2. State Transition
Trace what happens during lifecycle events:
- Fresh boot / cold start -- is all state initialized before first use?
- Configuration or key change at runtime -- which consumers get updated? Which are missed?
- Restart / reconnect -- what persists? What is lost? What becomes stale?
- TTL or token expiry -- does state actually revert or clean up?
- Failover / degraded mode -- does the fallback path actually work?

Red flags: state surviving a context change it should not, cache not invalidated on config reload, downstream consumers not notified of upstream changes.

Example: An auth token is refreshed, but the WebSocket connection still uses the old token until the next reconnect.

### 3. Boundary Values / Input Normalization
Test edge inputs at every entry point:
- Empty string `""`, `null`, `undefined`, `NaN`, `Infinity`, `0`, `false`
- `"7xyz"` through `parseInt` (silently returns 7), `"  abc  "` through trim
- Hex strings with uppercase, odd length, or non-hex characters
- Domain-meaningful zeros (ID 0, lane 0, window 0, TTL 0 -- valid or sentinel?)
- Header spoofing (X-Forwarded-For, Host), unicode in identifiers

Red flags: `parseInt` without radix or NaN guard, `||` vs `??` for falsy-but-valid values (0, ""), missing length checks on buffers.

Example: `if (laneId)` skips lane 0 because 0 is falsy, but lane 0 is a valid assignment.

### 4. Security Posture
Check every auth / crypto / trust boundary:
- **Auth**: fail-open vs fail-closed when token is missing or empty
- **Storage**: are secrets in appropriate secure storage (Keychain, credential store) vs plaintext?
- **Replay protection**: nonce or sequence fences -- do they reset when they should not?
- **HMAC / signatures**: correct key, correct scope, constant-time comparison
- **CORS**: preflight leaking permissive headers on sensitive endpoints
- **Health endpoints**: reporting "ok" when the service cannot actually serve requests
- **Route matching**: `startsWith` vs exact match allowing path traversal

Red flags: health endpoint returning 200 when critical dependencies are down, binary vs text WebSocket frame mismatch, secrets logged in debug mode.

Example: `/admin` route uses `path.startsWith('/admin')` which also matches `/administrator-panel`.

### 5. Cross-Component Consistency
When the same feature exists on multiple platforms or services, do they agree?
- Default values for shared configuration
- Error codes and their numeric values
- Behavioral ladders (rate-based vs count-based thresholds)
- Deduplication logic (which components have it? Same TTL?)
- Telemetry field names and types
- Health/status response shapes

Red flags: one platform defaults a feature to `true` while another defaults to `false`, enum values diverging across languages, timestamp units (seconds vs milliseconds) disagreeing.

Example: The JS client sends timestamps in milliseconds, but the Go server expects seconds. Works fine until the value is used for expiry math.

### 6. Dead Code / False Interface
Find things that LOOK functional but are not wired up:
- UI element exists but has no event handler
- Config field is stored/persisted but never read back
- Method is exported but never imported anywhere
- Tests exist but all cases are conditionally skipped (silent 12/12 skip)
- README documents a feature that throws "not implemented" at runtime
- Optional chaining masks a missing method (`store.getItem?.()` silently returns undefined)
- Status indicator shows success but no actual verification occurred

Red flags: `assert.ok(true)` (always passes), inline test copies instead of production imports, provider/adapter that always throws.

Example: A "proof" badge shows green in the UI, but the proof collection endpoint is stubbed and always returns `{ status: "ok" }`.

### 7. Test Gaps
Beyond "is there a test?" -- check test quality and coverage distribution:
- Does the test import the **production** module or a local copy of the logic?
- Does the test actually assert the behavior it claims to verify?
- Are there tests that always skip due to environment or config checks?
- Are critical paths (auth, state change, sync, failover) tested end-to-end?
- Does the test catch regressions if the implementation changes?
- **Layer distribution**: count tests per layer. If the store has 14 tests but the HTTP layer has 0, that is a gap.

Red flags: test file testing an inline class instead of the production module, all assertions passing because they test constants, zero integration tests for a multi-service system.

Example: `auth.test.js` has 20 unit tests for token parsing but zero tests for the HTTP middleware that actually enforces auth on routes.

## Output Format

For each finding:
```
Lens: [which of the 7]
Target: [file:line or component]
Contract: [what is expected]
Reality: [what actually happens]
Severity: P1 (data loss / security) | P2 (wrong behavior) | P3 (quality / maintainability) | P4 (cosmetic / docs)
Evidence: [specific code reference or snippet]
```

Group findings by severity (P1 first), then by lens within each severity group.

At the end, provide a summary table:

```
| Severity | Count | Top Lens |
|----------|-------|----------|
| P1       | N     | ...      |
| P2       | N     | ...      |
| P3       | N     | ...      |
| P4       | N     | ...      |
```

## When to Use Each Lens

| Situation | Primary lens | Supporting lenses |
|-----------|-------------|-------------------|
| New feature review | CITR | Security, Test gaps |
| Post-incident review | State transition | Security, Boundary |
| Pre-release audit | All 7 | -- |
| Documentation review | CITR | Dead code, Consistency |
| Security review | Security | Boundary, State transition |
| Refactoring review | Dead code | CITR, Test gaps |
| Multi-platform feature | Consistency | Boundary, CITR |
| CI/test reliability | Test gaps | Dead code, CITR |
