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

## Installation

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

**28 rules** organized into two tiers. Full details in [`SKILL.md`](SKILL.md).

### Tier 1: Code-Detectable (10 rules)

D1 (secrets) · D2 (PII logging) · D3 (insecure storage) · D4 (SQLi) · D5 (XSS) · D6 (eval) · D7 (auth) · D8 (dark patterns) · D9 (accessibility) · D10 (error handling)

### Tier 2: Context-Based Guidance (18 rules)

Data protection (C1-C3), communication (C4-C5), compliance (C6), quality (C7-C9), judgment (C10-C12), management (C13-C14), collaboration (C15-C16), growth (C17-C18).

Full details in [`references/`](references/).

## Examples

See `examples/` for full walkthroughs:
- Code review with PII leak → D2 remediation
- Analytics privacy → C1-C3 guidance
- Accessibility audit → D9 remediation

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
