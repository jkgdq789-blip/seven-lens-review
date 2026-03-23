# Seven-Lens Review -- 7-Lens Code Review Skill for Claude Code

A structured code review methodology that applies 7 overlapping analytical lenses to find bugs that conventional review misses. Built for use as a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash command.

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

## How It Was Developed

This skill was refined over **1,300 cycles** of real-world code review on a multi-platform event gate system spanning iOS (Swift), Android (Kotlin), browser PWA (JS), Node.js servers, JVM kiosk apps, LAN relays, and Web3 bridges.

## Real Results

- **138 bugs found**, including multiple P1 severity issues
- Caught bugs that **8 rounds of conventional single-file review missed**
- Most effective at finding cross-component inconsistencies and integration boundary failures

## Installation

Copy `SKILL.md` to your Claude Code commands directory:

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
/seven-lens-review src/api/routes.ts — focus on input validation
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

## License

MIT
