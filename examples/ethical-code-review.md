# Example: The Code That Passes Every Check

## The scenario

A developer submits a PR for the new subscription flow. You run the usual checks:

- ESLint: **0 errors**
- Snyk: **0 vulnerabilities**
- TypeScript: **0 type errors**
- Unit tests: **passing**
- Two PR approvals: **done**

The code is clean. Here it is:

```tsx
// SubscriptionModal.tsx
export function SubscriptionModal({ plan }: { plan: Plan }) {
  const [opted, setOpted] = useState(true);         // marketing emails
  const [trial, setTrial] = useState(false);
  const deadlineTs = Date.now() + 1000 * 60 * 30;  // 30-min "offer"

  return (
    <Modal>
      <h2>Start your {plan.name} plan</h2>

      <Countdown deadline={deadlineTs} label="Offer expires in" />

      <form onSubmit={handleSubmit}>
        <label>
          <input
            type="checkbox"
            checked={opted}
            onChange={e => setOpted(e.target.checked)}
          />
          Send me product updates and offers (recommended)
        </label>

        <button type="submit">
          Start free trial
        </button>

        <span
          style={{ fontSize: '10px', color: '#aaa' }}
          onClick={() => setShowCancelInfo(true)}
        >
          cancellation info
        </span>
      </form>
    </Modal>
  );
}
```

No scanner will flag this. The code is correct. The patterns are the problem.

---

## deon review

### Ethical Review

| # | Issue | Rule | Severity |
|---|-------|------|----------|
| 1 | Marketing checkbox pre-checked with "recommended" label — consent manufactured by default | **D8** | High |
| 2 | Fake countdown timer for a non-expiring offer — manufactured urgency | **D8** | High |
| 3 | Cancellation info hidden in 10px grey text, outside the button flow | **D8** | High |

### Why this matters

These patterns are not bugs. They are deliberate design choices that extract value from users by bypassing their rational decision-making. They are:

- **Illegal** in many jurisdictions. GDPR Art. 7 requires consent to be as easy to withdraw as to give. Pre-checked boxes fail this test. Several EU regulators have issued fines specifically for this pattern.
- **Deceptive** under FTC guidelines (US) and Consumer Protection from Unfair Trading Regulations (UK).
- **Harmful to trust.** Users who discover these patterns don't forget.

### Checklist

- [ ] D8: No pre-checked consent boxes
- [ ] D8: No artificial urgency (timers for non-expiring offers)
- [ ] D8: Cancellation as visible and easy as subscription

### Remediation

#### 1. Pre-checked consent → Explicit opt-in

```tsx
// BAD: Consent manufactured by default
const [opted, setOpted] = useState(true);

// GOOD: Explicit opt-in
const [opted, setOpted] = useState(false);
```

And remove "recommended" — that's social pressure on a consent choice.

```tsx
// BAD
<label>
  <input type="checkbox" checked={opted} ... />
  Send me product updates and offers (recommended)
</label>

// GOOD
<label>
  <input type="checkbox" checked={opted} ... />
  Send me product updates and offers
</label>
```

#### 2. Remove the fake countdown

If the offer genuinely expires, show the real date. If it doesn't expire, remove the timer entirely.

```tsx
// BAD: Manufactured urgency
<Countdown deadline={Date.now() + 1000 * 60 * 30} label="Offer expires in" />

// GOOD: Only show if real
{plan.expiresAt && (
  <p>This offer is available until {formatDate(plan.expiresAt)}.</p>
)}
```

#### 3. Cancellation as visible as subscription

```tsx
// BAD: Hidden in 10px grey text
<span style={{ fontSize: '10px', color: '#aaa' }} onClick={...}>
  cancellation info
</span>

// GOOD: A real link at the same visual weight as the CTA
<button type="submit">Start free trial</button>
<a href="/cancel-policy" className="cancel-link">
  Cancel anytime — here's how
</a>
```

---

### The clean version

```tsx
export function SubscriptionModal({ plan }: { plan: Plan }) {
  const [emailOptIn, setEmailOptIn] = useState(false);

  return (
    <Modal>
      <h2>Start your {plan.name} plan</h2>

      {plan.expiresAt && (
        <p>This offer is available until {formatDate(plan.expiresAt)}.</p>
      )}

      <form onSubmit={handleSubmit}>
        <label>
          <input
            type="checkbox"
            checked={emailOptIn}
            onChange={e => setEmailOptIn(e.target.checked)}
          />
          Send me product updates and offers
        </label>

        <button type="submit">Start free trial</button>
        <a href="/cancel-policy" className="cancel-link">
          Cancel anytime — here's how
        </a>
      </form>
    </Modal>
  );
}
```

### Rules applied

| Rule | ACM/IEEE reference |
|------|-------------------|
| D8: Dark patterns | Principle 1: Public interest; Principle 2: Client and employer — do not deceive |
