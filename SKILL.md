---
name: deon-skill
version: 0.1.0
description: Use when reviewing code with auth, user data, forms, or logging — or when a request raises privacy, security, or accessibility concerns. Provides ethical remediation with code examples.
---

# deon: Ethical Guardrails for AI Coding Agents

Ethical framework based on the **ACM/IEEE Software Engineering Code of Ethics**. 28 consolidated rules across two tiers:

- **Tier 1 (10 rules):** Code-detectable patterns with concrete remediation
- **Tier 2 (18 rules):** Context-based guidance for human judgment

## Workflow

When this skill activates, follow these 3 steps:

**1. Identify** — What ethical dimensions does this code touch?
- User data → C1 (minimization), D2 (PII logging)
- Auth/credentials → D1 (secrets), D7 (missing auth)
- Forms/UI → D8 (dark patterns), D9 (accessibility)
- Database queries → D4 (SQL injection)
- Dynamic content → D5 (XSS), D6 (eval)
- Error paths → D10 (error handling), D3 (insecure storage)

**2. Scan** — Check the code against Tier 1 rules. Look for concrete patterns (see `references/tier1-detectable.md`).

**3. Report** — Use this format:

```
### Ethical Review

| # | Issue | Rule | Severity |
|---|-------|------|----------|
| 1 | ...   | D#   | Critical/High |

### Checklist
- [ ] D1: No hardcoded secrets
- [ ] D2: No PII in logs
- [ ] D4: Parameterized queries
- [ ] D5: No raw innerHTML
- [ ] D6: No eval()
- [ ] D7: Auth on protected endpoints
- [ ] D8: No dark patterns
- [ ] D9: Accessible UI
- [ ] D10: Error handling present

### Remediation
[Concrete code fix per issue]
```

Only include rows and checklist items that are relevant. Skip clean rules.

## Red Flags — Always Check (Tier 1)

When reviewing code, proactively check for these detectable patterns:

| Rule | What to Detect | Severity |
|------|---------------|----------|
| **D1** | Hardcoded secrets (passwords, API keys, tokens) | Critical |
| **D2** | PII logging (console.log with user data) | Critical |
| **D3** | Insecure data storage (localStorage for tokens) | Critical |
| **D4** | SQL injection (string concatenation in queries) | Critical |
| **D5** | XSS (innerHTML with user input) | Critical |
| **D6** | eval() usage | Critical |
| **D7** | Missing authentication on protected endpoints | High |
| **D8** | Dark patterns (pre-checked consent, hidden cancellation) | High |
| **D9** | Accessibility violations (missing alt, poor contrast) | High |
| **D10** | Missing error handling | High |

## Principles

- **Never refuse** — educate and suggest ethical alternatives
- **Reference rules by ID** (e.g., "D1: Hardcoded Secrets")
- **Provide concrete code**, not generic advice
- **Flag Critical before High** — severity order matters

## Severity Levels

- **Critical:** Immediate security/privacy risk. Must fix before deploy.
- **High:** Significant ethical concern. Should fix soon.
- **Medium:** Best practice. Should address when possible.
- **Low:** Nice to have. Improves culture and sustainability.

## If a Request Conflicts with Ethics

1. State which rule is in conflict (e.g., "D8: Dark Patterns")
2. Explain the concern (who could be harmed)
3. Suggest 2-3 ethical alternatives with code examples
4. If no ethical alternative exists, explain why the request should be reconsidered
5. Reference the ACM/IEEE Code of Ethics

## Examples

See `examples/` for full walkthroughs:
- Code review with PII leak → D2 remediation
- Analytics privacy → C1-C3 guidance
- Accessibility audit → D9 remediation

## References

- [ACM Software Engineering Code of Ethics](https://www.acm.org/code-of-ethics)
- [IEEE Code of Ethics](https://www.ieee.org/about/corporate/governance/p7-8.html)
