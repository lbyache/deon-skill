# Example: Privacy by Design

## Scenario

Building a user analytics feature that tracks page views and user behavior.

## Applying deon

### Step 1: Identify ethical dimensions

This feature involves:
- **User data collection** → **C1** (Data minimization)
- **Potential for tracking without consent** → **C2** (Explicit consent)
- **Data retention** → **C3** (Data retention policy)
- **Logging user data** → **D2** (PII logging)

### Step 2: Load relevant rules

Reference: `references/tier1-detectable.md` (D2) and `references/tier2-context.md` (C1-C3)

### Step 3: Design with privacy built-in

#### Bad approach (privacy as afterthought)

```javascript
// BAD: Collect everything, figure out privacy later
function trackPageView(user, page, eventData) {
  analytics.track({
    userId: user.id,
    email: user.email,
    name: user.name,
    page: page,
    eventData: eventData,
    timestamp: Date.now(),
    ip: user.ip,
    userAgent: user.userAgent,
    location: user.location,
    // ... collect everything just in case
  });
}
```

**Issues:**
- **C1:** Excessive data collection (email, name, IP, location)
- **D2:** Logging PII (email, name)
- **C2:** No consent mechanism
- **C3:** No retention policy

#### Good approach (privacy by design)

```javascript
// GOOD: Data minimization, consent, anonymization
const ANALYTICS_CONFIG = {
  retentionDays: 90,
  collectPII: false,
  anonymizeIP: true,
};

function trackPageView(sessionId, page, eventData) {
  // C1: Only collect what's needed
  analytics.track({
    sessionId: anonymize(sessionId),  // Not directly identifiable
    page: page,
    eventData: sanitize(eventData),   // Remove any PII
    timestamp: Date.now(),
    // No email, name, IP, or location
  });
}

// C3: Auto-delete old data
async function cleanupOldData() {
  const cutoff = Date.now() - (ANALYTICS_CONFIG.retentionDays * 24 * 60 * 60 * 1000);
  await analytics.deleteBefore(cutoff);
}
```

### Privacy Checklist

- [x] **C1 - Data minimization**: Only collect sessionId, page, and sanitized event data
- [x] **C2 - Explicit consent**: Cookie banner with opt-in for analytics
- [x] **D2 - No PII logging**: Session IDs are hashed, no email/name logged
- [x] **C3 - Data retention**: Auto-delete after 90 days
- [x] **User control**: Users can opt out at any time
- [x] **Transparency**: Privacy policy explains what is collected and why

### Consent Implementation (C2)

```javascript
// Cookie consent banner
function getConsent() {
  return localStorage.getItem('analytics_consent') === 'true';
}

function trackPageView(sessionId, page, eventData) {
  // C2: Don't track without consent
  if (!getConsent()) {
    return;
  }
  
  analytics.track({
    sessionId: anonymize(sessionId),
    page: page,
    eventData: sanitize(eventData),
    timestamp: Date.now(),
  });
}
```

### Rules Applied

| Rule | How Applied |
|------|-------------|
| **D2** | No PII logging: session IDs anonymized, no email/name in logs |
| **C1** | Data minimization: only sessionId, page, sanitized eventData |
| **C2** | Explicit consent: opt-in required before tracking |
| **C3** | Retention policy: auto-delete after 90 days |
