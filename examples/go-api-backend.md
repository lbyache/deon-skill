# Example: The Go API That Ships Dark Patterns as a Contract (Go)

## The scenario

A backend engineer builds the signup API for a new SaaS product. The code is idiomatic Go: proper error handling, context propagation, structured logging with `slog`, no hardcoded credentials. `gosec` finds nothing.

```go
// handlers/signup.go
package handlers

import (
	"encoding/json"
	"log/slog"
	"net/http"
	"time"

	"github.com/acme/app/db"
	"github.com/acme/app/email"
)

type SignupRequest struct {
	Email     string `json:"email"`
	Password  string `json:"password"`
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
	Company   string `json:"company"`
	Phone     string `json:"phone"`
	JobTitle  string `json:"job_title"`
}

type SignupResponse struct {
	UserID              int    `json:"user_id"`
	MarketingOptIn      bool   `json:"marketing_opt_in"`       // default: true
	ThirdPartySharing   bool   `json:"third_party_sharing"`    // default: true
	TrialEndsAt         string `json:"trial_ends_at"`
	UpgradeUrgencyMsg   string `json:"upgrade_urgency_msg"`
}

func (h *Handler) Signup(w http.ResponseWriter, r *http.Request) {
	var req SignupRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "invalid request", http.StatusBadRequest)
		return
	}

	slog.Info("new signup", "email", req.Email, "company", req.Company,
		"phone", req.Phone, "job_title", req.JobTitle)

	user, err := h.DB.CreateUser(r.Context(), db.CreateUserParams{
		Email:     req.Email,
		Password:  req.Password, // stored before hashing check
		FirstName: req.FirstName,
		LastName:  req.LastName,
		Company:   req.Company,
		Phone:     req.Phone,
		JobTitle:  req.JobTitle,
		MarketingOptIn:    true,  // opt users in by default
		ThirdPartySharing: true,  // opt users in by default
	})
	if err != nil {
		slog.Error("failed to create user", "err", err)
		http.Error(w, "signup failed", http.StatusInternalServerError)
		return
	}

	email.SendWelcome(user.Email, user.FirstName)

	trialEnd := time.Now().Add(14 * 24 * time.Hour)

	json.NewEncoder(w).Encode(SignupResponse{
		UserID:            user.ID,
		MarketingOptIn:    true,
		ThirdPartySharing: true,
		TrialEndsAt:       trialEnd.Format(time.RFC3339),
		UpgradeUrgencyMsg: "⚡ Only 3 seats left at this price — upgrade before your trial ends!",
	})
}
```

`gosec`: **0 issues**. `staticcheck`: **0 issues**. Tests: **passing**.

---

## deon review

### Ethical Review

| # | Issue | Rule | Severity |
|---|-------|------|----------|
| 1 | `marketing_opt_in: true` and `third_party_sharing: true` set server-side without user action | **C2** | High |
| 2 | `upgrade_urgency_msg` is a hardcoded false scarcity claim | **D8** | High |
| 3 | `phone` and `job_title` collected at signup with no stated purpose | **C1** | High |
| 4 | `slog.Info` logs `email`, `phone`, `job_title` — PII in structured logs | **D2** | Critical |
| 5 | `Password` field passed to `CreateUser` before any hashing — surface area for accidental plaintext storage | **D3** | Critical |

### Why this matters

**On C2 (consent defaults):** Setting `marketing_opt_in` and `third_party_sharing` to `true` server-side means the API is the consent mechanism — and it's opting users in without any affirmative act. Under GDPR, this fails the "freely given, specific, informed, unambiguous" test. Under CCPA, selling/sharing data without opt-out violates §1798.120.

**On D8 (fake scarcity):** `"Only 3 seats left at this price"` is a static string in source code. It is never true. This is a deceptive trade practice under FTC guidelines and equivalent consumer protection law in most jurisdictions.

**On C1 (data minimization):** `phone` and `job_title` are collected at signup but not used anywhere in this flow. Every field collected without purpose is GDPR liability with no product benefit.

### Checklist

- [ ] C2: Consent fields default to `false` — user must opt in explicitly
- [ ] D8: No hardcoded false scarcity messages
- [ ] C1: Remove fields not used in the signup flow
- [ ] D2: No PII in structured logs
- [ ] D3: Password hashed before reaching the DB layer

### Remediation

#### 1. Consent defaults to false — always

```go
// BAD: Opt users in server-side
type CreateUserParams struct {
    MarketingOptIn:    true,
    ThirdPartySharing: true,
}

// GOOD: Only record what the user explicitly chose
type SignupRequest struct {
    Email          string `json:"email"`
    Password       string `json:"password"`
    FirstName      string `json:"first_name"`
    LastName       string `json:"last_name"`
    MarketingOptIn bool   `json:"marketing_opt_in"` // sent from UI, unchecked by default
}
```

The frontend checkbox is unchecked. The user checks it. That value — `false` by default — flows through to the DB. The server never overrides it.

#### 2. Remove the fake urgency message

```go
// BAD: Static lie
UpgradeUrgencyMsg: "⚡ Only 3 seats left at this price — upgrade before your trial ends!",

// GOOD: If you want urgency, use the real trial end date
// The date is already in the response. The frontend can render "X days left".
// Remove UpgradeUrgencyMsg from the response contract entirely.
```

#### 3. Collect only what you use at signup

```go
// BAD: Collecting phone and job title with no stated purpose
type SignupRequest struct {
    // ...
    Phone    string `json:"phone"`
    JobTitle string `json:"job_title"`
}

// GOOD: Ask for these later, in context, when they're needed
// e.g., phone at 2FA setup, job title in profile completion
type SignupRequest struct {
    Email     string `json:"email"`
    Password  string `json:"password"`
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
}
```

#### 4. Safe structured logging

```go
// BAD: PII in slog fields
slog.Info("new signup",
    "email", req.Email,
    "phone", req.Phone,
    "job_title", req.JobTitle,
)

// GOOD: Log only what's needed for debugging — never identity fields
slog.Info("new signup",
    "user_id", user.ID,
    "signup_method", "email",
)
```

#### 5. Hash password before it crosses any boundary

```go
// BAD: Raw password passed to DB layer
h.DB.CreateUser(r.Context(), db.CreateUserParams{
    Password: req.Password,
})

// GOOD: Hash at the handler boundary, before any logging or passing down
hashedPassword, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
if err != nil {
    http.Error(w, "signup failed", http.StatusInternalServerError)
    return
}
h.DB.CreateUser(r.Context(), db.CreateUserParams{
    PasswordHash: string(hashedPassword),
})
```

---

### The clean version

```go
// handlers/signup.go
package handlers

import (
	"encoding/json"
	"log/slog"
	"net/http"
	"time"

	"golang.org/x/crypto/bcrypt"

	"github.com/acme/app/db"
	"github.com/acme/app/email"
)

// C1: Only what signup actually needs
type SignupRequest struct {
	Email          string `json:"email"`
	Password       string `json:"password"`
	FirstName      string `json:"first_name"`
	LastName       string `json:"last_name"`
	MarketingOptIn bool   `json:"marketing_opt_in"` // UI sends false by default
}

type SignupResponse struct {
	UserID      int    `json:"user_id"`
	TrialEndsAt string `json:"trial_ends_at"`
}

func (h *Handler) Signup(w http.ResponseWriter, r *http.Request) {
	var req SignupRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "invalid request", http.StatusBadRequest)
		return
	}

	// D3: Hash before it crosses any boundary
	hash, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
	if err != nil {
		http.Error(w, "signup failed", http.StatusInternalServerError)
		return
	}

	user, err := h.DB.CreateUser(r.Context(), db.CreateUserParams{
		Email:          req.Email,
		PasswordHash:   string(hash),
		FirstName:      req.FirstName,
		LastName:       req.LastName,
		MarketingOptIn: req.MarketingOptIn, // C2: user's explicit choice, not server default
	})
	if err != nil {
		slog.Error("failed to create user", "err", err)
		http.Error(w, "signup failed", http.StatusInternalServerError)
		return
	}

	// D2: No PII in logs
	slog.Info("new signup", "user_id", user.ID)

	email.SendWelcome(user.Email, user.FirstName)

	json.NewEncoder(w).Encode(SignupResponse{
		UserID:      user.ID,
		TrialEndsAt: time.Now().Add(14 * 24 * time.Hour).Format(time.RFC3339),
		// D8: No fake urgency — the client can compute "X days left" from TrialEndsAt
	})
}
```

### Rules applied

| Rule | What it caught |
|------|---------------|
| C2: Explicit consent | Consent fields defaulted to `true` server-side |
| D8: Dark patterns | Hardcoded false scarcity message in API response |
| C1: Data minimization | `phone` and `job_title` collected without use |
| D2: PII logging | Identity fields written to structured logs |
| D3: Insecure storage | Password passed unhashed to DB layer |

### ACM/IEEE reference

> *"Software engineers shall act in a manner that is in the best interests of their client and employer, consistent with the public interest."*
> — ACM/IEEE Software Engineering Code of Ethics, Principle 2
