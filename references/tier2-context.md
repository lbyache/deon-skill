# Tier 2: Context-Required Rules

These rules require human judgment. The agent suggests best practices but cannot detect violations automatically.

Rules are organized by whether they're detectable from a feature request (design-time) or from existing code (review-time).

## User Rights (Design-time — check before building)

| Rule | What to check | Severity |
|------|--------------|----------|
| **C1** | Data minimization — is every field you're collecting necessary for the stated purpose? | High |
| **C2** | Explicit consent — is tracking/collection opt-in? Is consent granular and revocable? | High |
| **C3** | Data retention — is there a defined deletion period? Can users delete their own data? | Medium |
| **C4** | Transparency — do users know what's collected and why, in plain language? | Medium |

## Professional Integrity (Review-time — check when making recommendations)

| Rule | What to check | Severity |
|------|--------------|----------|
| **C6** | License compliance — are all dependencies' licenses compatible? Are authors attributed? | Medium |
| **C7** | Test coverage — is business logic covered? Are error paths tested? | Medium |
| **C8** | Code review — does security/auth code have at least two reviewers? | Medium |
| **C9** | Documentation — are public APIs, critical decisions, and non-obvious logic documented? | Medium |
| **C10** | Independent judgment — are recommendations based on technical merit, not convenience? | Medium |
| **C11** | Conflict disclosure — are biases or conflicts of interest acknowledged when advising? | Medium |
| **C12** | Honest representation — are limitations acknowledged? No exaggerated capability claims? | Medium |

---

**Removed from Tier 2:** C5 (deadline estimation), C13 (management culture), C14 (task assignment), C15 (collaboration tone), C16 (knowledge sharing), C17 (continuous learning), C18 (work-life balance). These are valid professional values but belong in a team handbook, not a code ethics guardrail. Keeping them here diluted the signal for rules that actually apply to software artifacts.
