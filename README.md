```diff
+ ┌───┐┌───┐┌───┐┌───┐
+ │ d ││ e ││ o ││ n │
+ └───┘└───┘└───┘└───┘
```

# deon

**Ethical guardrails for AI coding agents**

![agent-skills](https://img.shields.io/badge/agent--skills-purple)
![ai-ethics](https://img.shields.io/badge/ai--ethics-blue)
![responsible-ai](https://img.shields.io/badge/responsible--ai-green)
![secure-coding](https://img.shields.io/badge/secure--coding-red)
![claude-code](https://img.shields.io/badge/claude--code-242424)
![opencode](https://img.shields.io/badge/opencode-orange)
![cursor](https://img.shields.io/badge/cursor-cyan)

> *"Software engineers shall act consistently with the public interest."*
> — ACM/IEEE Software Engineering Code of Ethics

> **Seed project** — This is my small contribution to making agentic AI responsible from day one.
> Early, opinionated, and meant to evolve. Fork it, break it, PR it.

## The gap deon fills

Your linter passed. Your SAST scan passed. The PR has two approvals.

And yet:

- The marketing checkbox is pre-checked
- The analytics tracker collects location without telling the user
- The cancellation flow is buried 5 screens deep
- The form works perfectly with a mouse and is invisible to a screen reader

No tool flagged any of this. Because these aren't bugs — they're **ethical failures**. deon makes them visible.

## Example

**You:** *"Make the subscription checkbox pre-checked and hide the cancellation flow."*

**Agent (with deon):**

> **Ethical concern** — D8: Dark patterns
>
> Pre-checked consent and hidden cancellation are dark patterns. Legally actionable under GDPR/FTC.
>
> **Suggested approach:**
> 1. Unchecked by default — explicit opt-in
> 2. Cancellation as easy as subscription
> 3. Reminder email before renewal

## What deon is NOT

- **Not a security scanner.** Secrets, SQLi, and XSS are already covered by linters, Semgrep, Snyk, and SonarQube. deon repeats them as a last line of defense — but that's not its purpose.
- **Not a linter.** It doesn't run on every file automatically. It activates on ethical dimensions.
- **Not a style guide.** deon doesn't care about naming conventions or code organization.

## What deon IS

deon is the enforcer for the gap between *"the code works and passes all checks"* and *"the code respects the people who use it."*

Its unique territory — the rules no other tool covers:

| Rule | What it catches |
|------|----------------|
| **D8** | Dark patterns: pre-checked consent, hidden cancellation, fake urgency |
| **D9** | Accessibility as an ethical obligation, not just a technical checkbox |
| **C1** | Collecting more data than you need |
| **C2** | Opt-out instead of opt-in — stealing consent by default |
| **C3** | Storing data indefinitely with no retention policy |

## Installation

```bash
npx skills add LauYache/deon-skill
```

Or manually:

```bash
git clone https://github.com/LauYache/deon-skill.git
cp -r deon-skill ~/.config/opencode/skills/deon-skill
```

For other agents, replace `~/.config/opencode/skills/` with:
- **Claude Code:** `.claude/skills/`
- **Cursor:** `.cursor/skills/`

## Usage

Invoke with `/deon` or `skill:deon` (varies by agent).

The skill detects patterns from **Tier 1 rules** (D1-D10) and suggests remediation with code examples. For **Tier 2 rules** (C1-C18), it provides guidance when relevant.

## Rules

**26 rules** organized into two tiers. Full details in [`SKILL.md`](SKILL.md).

### Tier 1: Code-Detectable (10 rules)

Priority (deon's unique territory): **D8** (dark patterns) · **D9** (accessibility)

Secondary (last line of defense): D1 (secrets) · D2 (PII logging) · D3 (insecure storage) · D4 (SQLi) · D5 (XSS) · D6 (eval) · D7 (auth) · D10 (error handling)

### Tier 2: Context-Based Guidance (16 rules)

User rights — C1 (minimization) · C2 (consent) · C3 (retention) · C4 (transparency)

Professional integrity — C6 (licenses) · C7 (test coverage) · C8 (code review) · C9 (documentation) · C10 (judgment) · C11 (conflicts) · C12 (honest representation)

Full details in [`references/`](references/).

## Examples

See `examples/` for full walkthroughs:

| Example | Stack | What it shows |
|---------|-------|--------------|
| [dark-pattern-request.md](examples/dark-pattern-request.md) | — | **The core case.** A request is intercepted before any code is written. No linter could catch this. |
| [ethical-code-review.md](examples/ethical-code-review.md) | TypeScript / React | Code that passes ESLint, Snyk, TS, and two PR reviews — and still has three ethical violations. |
| [go-api-backend.md](examples/go-api-backend.md) | Go | A signup API that passes `gosec` and `staticcheck` — with consent stolen by default and a hardcoded lie in the response contract. |
| [python-ml-pipeline.md](examples/python-ml-pipeline.md) | Python / scikit-learn | A churn model that passes `bandit` and `ruff` — using PII as features and logging full user records. |
| [privacy-by-design.md](examples/privacy-by-design.md) | JavaScript | Building an analytics feature with C1-C3 applied from the start. |
| [accessibility-audit.md](examples/accessibility-audit.md) | HTML | D9 in practice: auditing a landing page for accessibility as an ethical obligation. |

## Structure

```
deon-skill/
├── SKILL.md              # Skill definition
```

## License

MIT — see [LICENSE](LICENSE).

## Tags

`agent-skills` `ai-ethics` `responsible-ai` `secure-coding` `claude-code` `opencode` `cursor`

## Roadmap

| What | Why |
|------|-----|
| Tier 2 cleanup | C5, C13, C17, C18 are management advice, not code ethics — dilute signal |
| Python + Go examples | Currently JS-only — limits relevance for other stacks |
| CHANGELOG.md | Track versions so users know when to update |
| Test harness | Feed known-bad code, verify the agent flags the right rules |

Open to PRs on any of these.

## References

- [ACM Software Engineering Code of Ethics](https://www.acm.org/code-of-ethics)
- [IEEE Code of Ethics](https://www.ieee.org/about/corporate/governance/p7-8.html)
