# Example: Accessibility Audit

## Scenario

Reviewing a landing page for accessibility compliance.

## Applying deon

### Step 1: Identify ethical dimensions

This involves:
- **Accessibility for disabled users** → **D9** (Accessibility violations)

### Step 2: Load relevant rule

Reference: `references/tier1-detectable.md` — D9 (Accessibility violations)

### Step 3: Audit the page

#### Original page (with issues)

```html
<!-- BAD: Multiple accessibility issues -->
<div class="hero">
  <img src="hero-banner.jpg">
  <h1>Welcome to Our Platform</h1>
  <p style="color: #cccccc; background-color: #ffffff;">
    The best solution for your needs. Click below to get started!
  </p>
  <button onclick="navigate('/signup')">Sign Up</button>
  <button onclick="navigate('/login')">Login</button>
</div>

<div class="form">
  <input type="email" placeholder="Enter email">
  <input type="password" placeholder="Password">
  <button onclick="submit()">Submit</button>
</div>

<video src="demo.mp4" autoplay></video>
```

#### Issues Found (D9: Accessibility Violations)

| # | Issue | WCAG | Severity |
|---|-------|------|----------|
| 1 | Image without alt text | 1.1.1 | High |
| 2 | Low color contrast (#ccc on #fff = 1.61:1, needs 4.5:1) | 1.4.3 | High |
| 3 | Buttons without accessible names | 4.1.2 | Medium |
| 4 | Form inputs without labels | 1.3.1 | High |
| 5 | Video autoplay without controls | 1.4.2 | Medium |
| 6 | No keyboard navigation | 2.1.1 | High |

#### Fixed page

```html
<!-- GOOD: Accessible -->
<div class="hero" role="banner">
  <img src="hero-banner.jpg" alt="Person using our platform on a laptop, smiling">
  <h1>Welcome to Our Platform</h1>
  <p style="color: #333333; background-color: #ffffff;">
    The best solution for your needs. Click below to get started!
  </p>
  <a href="/signup" class="btn btn-primary">Sign Up</a>
  <a href="/login" class="btn btn-secondary">Login</a>
</div>

<div class="form">
  <form onsubmit="handleSubmit(event)">
    <label for="email">Email address</label>
    <input type="email" id="email" name="email" required aria-describedby="email-help">
    <span id="email-help" class="sr-only">We'll never share your email.</span>
    
    <label for="password">Password</label>
    <input type="password" id="password" name="password" required minlength="8">
    
    <button type="submit">Create account</button>
  </form>
</div>

<video src="demo.mp4" controls aria-label="Platform demo video">
  <track kind="captions" src="demo-captions.vtt" srclang="en" label="English">
</video>
```

### Accessibility Checklist (D9)

- [x] All images have descriptive alt text
- [x] Color contrast meets WCAG AA (4.5:1 for normal text, 3:1 for large text)
- [x] Interactive elements are keyboard accessible
- [x] Form inputs have proper labels
- [x] Content is screen reader compatible (semantic HTML, ARIA where needed)
- [x] No auto-playing media without controls
- [x] Video has captions
- [x] Focus indicators are visible
- [x] No content that flashes more than 3 times per second

### Rule Applied

**D9: Accessibility Violations** [High]

> Ensure all users can access and use your software, regardless of ability.

By ensuring accessibility, we prevent discrimination against users with:
- **Visual impairments** (screen reader support, alt text, contrast)
- **Motor impairments** (keyboard navigation)
- **Hearing impairments** (video captions)
- **Cognitive impairments** (clear labels, semantic structure)
