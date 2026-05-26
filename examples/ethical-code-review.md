# Example: Ethical Code Review

## Scenario

A developer submits a PR with the following code:

```javascript
app.post('/api/users', async (req, res) => {
  const user = req.body;
  
  // Store user
  db.users.insert(user);
  
  // Log for debugging
  console.log('New user:', JSON.stringify(user));
  
  // Send welcome email
  sendEmail(user.email, 'Welcome!');
  
  res.json({ success: true });
});
```

## Ethical Review Using deon

### Issues Found

| # | Issue | Rule | Severity |
|---|-------|------|----------|
| 1 | Logging full user object (may contain password, PII) | **D2** | Critical |
| 2 | No input validation | **D10** | High |
| 3 | No error handling | **D10** | High |
| 4 | No authentication | **D7** | High |
| 5 | No data minimization | **C1** | High |

### Review Comments

**1. PII Logging (D2)** [Critical]

Logging the full user object may expose sensitive data (password, PII) in logs.

**Fix:**
```javascript
// Only log non-sensitive fields
logger.info('User registered', { 
  id: user.id, 
  emailDomain: user.email.split('@')[1] 
});
```

**2. Missing Error Handling (D10)** [High]

No error handling. If the database insert fails, the user gets a success response.

**Fix:**
```javascript
try {
  const user = await db.users.insert(validatedData);
  res.json({ success: true, id: user.id });
} catch (error) {
  logger.error('User creation failed', { error: error.message });
  res.status(500).json({ error: 'Failed to create user' });
}
```

**3. Missing Authentication (D7)** [High]

No authentication on user creation endpoint. Anyone can create users.

**Fix:**
```javascript
app.post('/api/users', authenticate, async (req, res) => { 
  // ...
});
```

**4. Data Minimization (C1)** [High]

Collecting and storing full user object without validation. Only store what's needed.

**Fix:**
```javascript
const user = await db.users.insert({
  name: value.name,
  email: value.email,
  password: await hashPassword(value.password),
});
```

### Revised Code

```javascript
const Joi = require('joi');
const logger = require('./logger');

const userSchema = Joi.object({
  name: Joi.string().required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
});

app.post('/api/users', authenticate, async (req, res) => {
  try {
    const { error, value } = userSchema.validate(req.body);
    if (error) {
      return res.status(400).json({ error: error.message });
    }

    const user = await db.users.insert({
      name: value.name,
      email: value.email,
      password: await hashPassword(value.password),
    });

    // D2: Log only non-sensitive fields
    logger.info('User registered', { 
      id: user.id, 
      emailDomain: user.email.split('@')[1] 
    });
    await sendEmail(user.email, 'Welcome!');

    res.json({ success: true, id: user.id });
  } catch (error) {
    // D10: Error handling
    logger.error('User creation failed', { error: error.message });
    res.status(500).json({ error: 'Failed to create user' });
  }
});
```
