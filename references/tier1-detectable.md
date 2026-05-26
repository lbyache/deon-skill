# Tier 1: Code-Detectable Rules

These rules have concrete patterns an agent can detect and flag in code.

---

## D1. Hardcoded Secrets [Critical]

**Detection:** `password = "..."`, `apiKey = "sk-..."`, secrets in comments or connection strings

```javascript
// BAD
const API_KEY = "sk-1234567890abcdef";

// GOOD
const API_KEY = process.env.API_KEY;
```

---

## D2. PII Logging [Critical]

**Detection:** `console.log(user.password)`, `logger.info(JSON.stringify(user))`

```javascript
// BAD
console.log('User login:', { email: user.email, password: user.password });

// GOOD
console.log('User login:', { userId: user.id, timestamp: Date.now() });
```

---

## D3. Insecure Data Storage [Critical]

**Detection:** `localStorage.setItem('token', jwt)`, plaintext PII in DB

```javascript
// BAD
localStorage.setItem('authToken', jwt);

// GOOD: HttpOnly cookies
res.cookie('authToken', jwt, { httpOnly: true, secure: true, sameSite: 'strict' });
```

---

## D4. SQL Injection [Critical]

**Detection:** `"SELECT * FROM users WHERE id = " + userId`, template literals in SQL

```javascript
// BAD
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;

// GOOD
const query = "SELECT * FROM users WHERE email = $1";
db.query(query, [userEmail]);
```

---

## D5. XSS Vulnerabilities [Critical]

**Detection:** `element.innerHTML = userInput`, `dangerouslySetInnerHTML` without sanitization

```jsx
// BAD
<div dangerouslySetInnerHTML={{__html: userInput}} />

// GOOD
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{__html: DOMPurify.sanitize(userInput)}} />
```

---

## D6. eval() Usage [Critical]

**Detection:** `eval(userInput)`, `new Function(userInput)`, `setTimeout(stringCode)`

```javascript
// BAD
eval(userInput);

// GOOD
const data = JSON.parse(userInput);
```

---

## D7. Missing Authentication [High]

**Detection:** `app.post('/api/users', handler)` without auth middleware

```javascript
// BAD
app.post('/api/users', async (req, res) => {
  await createUser(req.body);
  res.json({ success: true });
});

// GOOD
app.post('/api/users', authenticate, async (req, res) => {
  await createUser(req.body);
  res.json({ success: true });
});
```

---

## D8. Dark Patterns [High]

**Detection:** Pre-checked consent, hidden cancellation, fake urgency

```jsx
// BAD
<input type="checkbox" defaultChecked={true} />

// GOOD
<input type="checkbox" defaultChecked={false} />
```

---

## D9. Accessibility Violations [High]

**Detection:** `<img>` without `alt`, contrast < 4.5:1, inputs without `<label>`

```html
<!-- BAD -->
<img src="logo.png">
<input type="email" placeholder="Email">

<!-- GOOD -->
<img src="logo.png" alt="Company logo">
<label for="email">Email address</label>
<input type="email" id="email">
```

---

## D10. Missing Error Handling [High]

**Detection:** Async operations without try/catch, DB ops without error states

```javascript
// BAD
app.post('/api/transfer', async (req, res) => {
  await db.transfer(req.body.from, req.body.to, req.body.amount);
  res.json({ success: true });
});

// GOOD
app.post('/api/transfer', async (req, res) => {
  try {
    const result = await db.transfer(req.body.from, req.body.to, req.body.amount);
    res.json({ success: true, txId: result.id });
  } catch (error) {
    logger.error('Transfer failed', { error: error.message });
    res.status(400).json({ error: 'Transfer failed' });
  }
});
```
