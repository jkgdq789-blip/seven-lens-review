[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Works with Claude Code](https://img.shields.io/badge/Works_with-Claude_Code-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)
[![GitHub stars](https://img.shields.io/github/stars/jkgdq789-blip/seven-lens-review?style=social)](https://github.com/jkgdq789-blip/seven-lens-review)

# Seven-Lens Review

7-lens code review skill for Claude Code -- finds bugs that conventional review misses.

## Quick Start

```bash
cp SKILL.md ~/.claude/commands/seven-lens-review.md   # install once
# then in Claude Code:
/seven-lens-review src/auth/                           # review any target
```

## Demo

```
$ /seven-lens-review src/api/rate-limiter.ts

  Scanning 12 files across 3 directories...

  BEFORE (conventional review): "Looks fine, rate limiter has tests."

  AFTER (seven-lens review):

  +------------------------------------------------------------------+
  | P1 | Boundary Values  | src/api/rate-limiter.ts:47               |
  |    | Contract: limit(reqsPerSec) rejects when rate exceeded       |
  |    | Reality:  parseInt(reqsPerSec) silently returns NaN for      |
  |    |           non-numeric input -- NaN < maxRate is false,       |
  |    |           so ALL requests pass through unchecked             |
  |    | Evidence: limit(req.headers['x-rate']) -- header is a string |
  +------------------------------------------------------------------+
  | P2 | State Transition  | src/api/rate-limiter.ts:83              |
  |    | Contract: window resets every 60s                            |
  |    | Reality:  clearInterval on config reload never fires because |
  |    |           timer ref is shadowed by local variable            |
  +------------------------------------------------------------------+
  | P3 | Test Gaps         | test/rate-limiter.test.ts                |
  |    | 14 unit tests for token bucket math, 0 tests for the HTTP   |
  |    | middleware that actually enforces rate limits on routes       |
  +------------------------------------------------------------------+

  Summary: 1 P1, 1 P2, 1 P3 across 3 lenses
```

## What It Does

Seven-Lens Review performs deep code review through 7 complementary perspectives applied simultaneously:

| # | Lens | What it catches |
|---|------|-----------------|
| 1 | **Contract-Implementation-Test-Reproduction (CITR)** | Docs promise X, code does Y, test checks Z |
| 2 | **State Transition** | Boot, reload, failover, TTL expiry -- state that leaks or goes stale |
| 3 | **Boundary Values** | 0, empty string, null, NaN, unicode, spoofed headers |
| 4 | **Security Posture** | Fail-open auth, replay gaps, HMAC scope, CORS leaks |
| 5 | **Cross-Component Consistency** | Same feature, different platforms, different defaults |
| 6 | **Dead Code / False Interface** | UI wired to nothing, config stored but never read, tests that silently skip |
| 7 | **Test Gap Analysis** | Tests that import their own copy, assert true, or skip under CI |

## Real Bug Found

A rate limiter accepted a `reqsPerSec` value from an HTTP header. The code used `parseInt(value)` without a NaN guard, then compared `NaN < maxRate`. Since that comparison is always `false`, the rate limiter silently allowed every request through -- including abuse traffic. Conventional review said "rate limiter has tests and passes." Seven-Lens Review caught it under the Boundary Values lens because it checks every entry point for NaN, 0, and empty-string behavior.

## How It Was Developed

This skill was refined over **1,300 cycles** of real-world code review on a multi-platform system spanning iOS (Swift), Android (Kotlin), browser PWA (JS), Node.js servers, JVM kiosk apps, LAN relays, and Web3 bridges.

### Real Results

- **138 bugs found**, including multiple P1 severity issues
- Caught bugs that **8 rounds of conventional single-file review missed**
- Most effective at finding cross-component inconsistencies and integration boundary failures

## Installation

```bash
# As a project command (scoped to one repo)
mkdir -p .claude/commands
cp SKILL.md .claude/commands/seven-lens-review.md

# As a user command (available in all projects)
mkdir -p ~/.claude/commands
cp SKILL.md ~/.claude/commands/seven-lens-review.md
```

## Usage

Inside Claude Code, invoke the skill with:

```
/seven-lens-review <target>
```

Examples:

```
/seven-lens-review the authentication middleware in src/auth/
/seven-lens-review the payment processing flow
/seven-lens-review all WebSocket message handlers
/seven-lens-review src/api/routes.ts -- focus on input validation
```

The skill works best when pointed at:
- A component or module directory
- A feature that spans multiple files
- A specific concern (e.g., "auth boundary", "state persistence")

## Output

Findings are grouped by severity (P1-P4), each with:
- Which lens detected it
- Target file and line
- What the contract/expectation is
- What actually happens
- Specific code evidence

## Compatible With

- **Claude Code** -- install as a slash command
- **Any language** -- JavaScript, TypeScript, Python, Go, Rust, Swift, Kotlin, Java, and more
- **Any multi-file codebase** -- monorepos, microservices, multi-platform projects

## License

[MIT](LICENSE)
