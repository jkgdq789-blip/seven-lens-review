[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Works with Claude Code](https://img.shields.io/badge/Works_with-Claude_Code-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)

# Seven-Lens Review -- Make AI code review actually thorough

AI reviews tend to check one thing at a time. This skill forces 7 overlapping checks in one pass.

## The problem

Claude Code reviews a file and says "looks good." But it only checked one angle -- maybe naming, maybe logic, maybe types. It did not check all of them together. Bugs hide in the gaps between single-pass checks.

Seven-Lens Review is a checklist that catches what single-pass review misses. Not a framework. Not a methodology. A checklist.

## Install in 10 seconds

```bash
cp SKILL.md ~/.claude/commands/seven-lens-review.md
```

Then in Claude Code:

```
/seven-lens-review src/auth/
```

## The 7 lenses

1. **Contract vs. implementation vs. tests** -- Docs promise X, code does Y, test checks Z. All three should agree.
2. **State transitions** -- Boot, reload, failover, TTL expiry. Does state leak or go stale?
3. **Boundary values** -- 0, empty string, null, NaN, max int. What happens at the edges?
4. **Security posture** -- Fail-open auth, replay gaps, HMAC scope, CORS misconfig.
5. **Cross-component consistency** -- Same feature on two platforms, two different defaults.
6. **Dead code and false interfaces** -- UI wired to nothing, config stored but never read.
7. **Test gap analysis** -- Tests that skip silently, assert true, or never test the HTTP layer.

## Example

```
/seven-lens-review src/api/rate-limiter.ts

  Single-pass review said: "Rate limiter has tests and passes."

  Seven-lens review found:

  P1 | Lens 3 (Boundary) | rate-limiter.ts:47
      parseInt(reqsPerSec) returns NaN for non-numeric header values.
      NaN < maxRate is always false. Every request passes through.

  P2 | Lens 2 (State)    | rate-limiter.ts:83
      clearInterval on config reload never fires -- timer ref
      shadowed by a local variable.

  P3 | Lens 7 (Test Gap) | rate-limiter.test.ts
      14 unit tests for token bucket math.
      0 tests for the HTTP middleware that enforces limits on routes.
```

## What it is

A skill file you drop into your Claude Code commands directory. When you invoke it, Claude reviews your target through all 7 lenses in a single pass instead of whatever single angle it would have picked on its own.

Works on any language. Works on any codebase. No dependencies.

## Installation

```bash
# Scoped to one repo
mkdir -p .claude/commands
cp SKILL.md .claude/commands/seven-lens-review.md

# Available everywhere
mkdir -p ~/.claude/commands
cp SKILL.md ~/.claude/commands/seven-lens-review.md
```

## Usage

```
/seven-lens-review src/auth/
/seven-lens-review the payment processing flow
/seven-lens-review all WebSocket message handlers
```

Point it at a directory, a feature, or a concern. It works best on anything that spans multiple files.

## License

[MIT](LICENSE)
