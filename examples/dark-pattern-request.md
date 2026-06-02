# Example: Intercepting a Dark Pattern Before the Code Is Written

## The scenario

This is deon's most important use case: the request itself is the problem. The developer hasn't written anything yet. A linter can't catch a request.

**The request:**

> "We need to increase newsletter signups. Can you make the opt-in checkbox pre-checked on the registration form? Also, make the 'unsubscribe' link in emails smaller and grey so it doesn't stand out."

---

## deon intercepts at request time

> **Ethical concern — D8: Dark Patterns**
>
> This request describes two dark patterns:
>
> **1. Pre-checked consent** (the checkbox)
> Checking the box on behalf of the user is manufactured consent. The user never chose to subscribe — they chose not to notice the checkbox.
>
> - Illegal under GDPR Art. 7 in the EU (explicit, affirmative action required)
> - Violates CAN-SPAM Act in the US (deceptive practices)
> - Common trigger for regulator action — several EU DPAs have issued fines specifically for pre-checked boxes
>
> **2. Obscured opt-out** (the small grey unsubscribe link)
> Making it harder to unsubscribe than to subscribe is an asymmetry that coerces continued consent. GDPR requires withdrawal to be "as easy as" giving consent.
>
> ---
>
> **Who gets harmed:** Users receive marketing emails they didn't consciously choose. Many won't know how to stop them. Those who figure it out will distrust your product.
>
> **What you actually want:** higher signup rates. Here are ethical paths to the same goal.

### Ethical alternatives

**Option 1: Better copy, not a pre-check**

The checkbox is unchecked, but the value proposition is clear:

```tsx
<label htmlFor="newsletter">
  <input type="checkbox" id="newsletter" name="newsletter" />
  Get weekly tips on [specific value]. Unsubscribe anytime.
</label>
```

Users who want it will check it. Your list will be smaller but engaged — which means better open rates, fewer spam complaints, and a healthier sender reputation.

**Option 2: Separate the newsletter from registration**

Don't ask at registration. Ask after the user has experienced value:

```tsx
// After first successful action in the app
<Banner>
  <p>Enjoying [feature]? Get weekly tips like this.</p>
  <button onClick={subscribeToNewsletter}>Yes, send me tips</button>
  <button onClick={dismissBanner}>No thanks</button>
</Banner>
```

Opt-in rates from contextual, post-value asks consistently outperform pre-checked registration boxes because the user has a reason to say yes.

**Option 3: Make unsubscribing a trust signal, not a liability**

A visible, easy unsubscribe link is not a risk — it's proof you respect the user.

```html
<!-- BAD: Hiding the exit -->
<a href="{{unsubscribeUrl}}" style="color: #ccc; font-size: 10px;">unsubscribe</a>

<!-- GOOD: Owning the exit -->
<a href="{{unsubscribeUrl}}" class="unsubscribe-link">
  Unsubscribe or manage preferences
</a>
```

Users who can leave easily are users who trust you enough to stay.

---

## Why deon catches this when nothing else does

At request time, there is no code to lint. There is no dependency to scan. There is no SQL to parameterize.

The only thing that can catch this is a tool that understands the ethical dimension of what's being asked — and has the framework to name it, explain it, and offer a better path.

That's deon's job.

### Rules applied

| Rule | What it caught |
|------|---------------|
| D8: Dark patterns | Pre-checked consent, obscured opt-out |
| C2: Explicit consent | Opt-in must be an affirmative act |

### ACM/IEEE reference

> *"Software engineers shall act in a manner that is in the best interests of... the public."*
> — ACM/IEEE Software Engineering Code of Ethics, Principle 1
