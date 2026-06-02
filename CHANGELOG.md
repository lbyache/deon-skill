# Changelog

All notable changes to this project will be documented in this file.

## [0.2.0] - 2026-06-02

### Changed
- **Rule reorganization**: Split Tier 1 into priority rules (D8, D9, C1-C3) and secondary rules (D1-D7, D10). Priority rules are deon's unique territory — what no linter catches.
- **Tier 2 cleanup**: Removed C5, C13-C18 (management/culture advice). Reorganized remaining rules into User Rights (design-time) and Professional Integrity (review-time). Total: 26 rules (down from 28).
- **Workflow update**: Added request-time interception — flag dark patterns before any code is written.
- **Checklist focus**: Shifted from security basics to ethical dimensions (D8, D9, C1-C3).

### Added
- **README sections**: "The gap deon fills", "What deon is NOT", "What deon IS" — clarifies deon's unique territory.
- **examples/dark-pattern-request.md**: Intercepting unethical requests before code exists.
- **examples/go-api-backend.md**: Signup API with server-side consent theft, fake scarcity, PII logging (Go).
- **examples/python-ml-pipeline.md**: ML model using PII as features, logging full user records (Python/scikit-learn).
- **examples/ethical-code-review.md**: Rewritten as dark pattern subscription modal case (TS/React).

---

## [0.1.0] - 2026-05-26

### Added
- Initial release: 28 rules across two tiers
- Tier 1: D1-D10 (code-detectable patterns)
- Tier 2: C1-C18 (context-based guidance)
- Basic workflow: Identify → Scan → Report
- examples/accessibility-audit.md, privacy-by-design.md, ethical-code-review.md
- references/tier1-detectable.md, tier2-context.md
- agents/ethical-reviewer.md
