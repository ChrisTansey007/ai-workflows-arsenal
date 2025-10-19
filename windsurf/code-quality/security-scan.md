---
id: wkf.security-scan
type: workflow
title: Security Scan
tags: [security, vulnerability, scanning, audit, compliance]
summary: Comprehensive security scanning of codebase, dependencies, and configurations to identify vulnerabilities and suggest automated remediation.
version: 1
---

# Security Scan

**Automated security vulnerability scanning and remediation**

## Description

This workflow performs comprehensive security scanning of your codebase, dependencies, and configurations to identify vulnerabilities, insecure patterns, and potential security risks.

## Use Case

Use this when:
- Before deploying to production
- After adding new dependencies
- Regular security audits
- Investigating potential vulnerabilities
- Preparing for security review

## Prerequisites

- Project dependencies installed
- Access to security scanning tools
- Node.js/npm or Python/pip (depending on stack)

## Usage

```bash
/security-scan
```

## Workflow Steps

### 1. Dependency Vulnerability Scan

Check for known vulnerabilities in dependencies:

**Node.js/npm:**
```bash
npm audit
npm audit --json > audit-report.json
```

**Python/pip:**
```bash
pip-audit
# or
safety check
```

**Ruby/gem:**
```bash
bundle audit
```

**Go:**
```bash
go list -json -m all | nancy sleuth
```

### 2. Analyze Audit Results

Parse and categorize findings:

```markdown
## Dependency Vulnerabilities

### Critical (Fix Immediately)
- [package-name] v[version]
  - CVE: CVE-2024-XXXXX
  - Severity: CRITICAL
  - Description: [vulnerability description]
  - Fix: Upgrade to v[safe-version]

### High (Fix Soon)
- [package-name] v[version]
  - CVE: CVE-2024-XXXXX
  - Severity: HIGH
  - Description: [vulnerability description]
  - Fix: Upgrade to v[safe-version]

### Medium & Low
- [Count] medium severity issues
- [Count] low severity issues
```

### 3. Code Security Scan

Scan code for security anti-patterns:

**What to check:**

#### A. Hardcoded Secrets
```regex
Search for:
- API keys: "api_key.*=.*['\"][a-zA-Z0-9]{20,}"
- Passwords: "password.*=.*['\"][^'\"]+['\"]"
- Tokens: "token.*=.*['\"][a-zA-Z0-9]{20,}"
- AWS keys: "AKIA[0-9A-Z]{16}"
- Private keys: "-----BEGIN.*PRIVATE KEY-----"
```

**Example findings:**
```markdown
🔴 CRITICAL: Hardcoded API key found
File: src/config/api.ts
Line: 15
Code: const API_KEY = "sk_live_12345abcdef"
Fix: Use environment variables
```

#### B. SQL Injection Risks
```javascript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ Safe
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
```

#### C. XSS Vulnerabilities
```javascript
// ❌ Vulnerable
element.innerHTML = userInput;

// ✅ Safe
element.textContent = userInput;
// or use DOMPurify
element.innerHTML = DOMPurify.sanitize(userInput);
```

#### D. Insecure Randomness
```javascript
// ❌ Weak
const token = Math.random().toString(36);

// ✅ Secure
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');
```

#### E. Path Traversal
```javascript
// ❌ Vulnerable
const file = req.query.file;
fs.readFile(`/uploads/${file}`);

// ✅ Safe
const path = require('path');
const safePath = path.normalize(file).replace(/^(\.\.[\/\\])+/, '');
```

### 4. Configuration Security Check

Review security configurations:

#### A. CORS Configuration
```javascript
// ❌ Too permissive
app.use(cors({ origin: '*' }));

// ✅ Restrictive
app.use(cors({ 
  origin: ['https://yourdomain.com'],
  credentials: true 
}));
```

#### B. Security Headers
Check for required headers:
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Strict-Transport-Security`
- `Content-Security-Policy`
- `X-XSS-Protection: 1; mode=block`

#### C. Authentication
```javascript
// ❌ Weak
if (user.password === inputPassword) {
  // login
}

// ✅ Secure
bcrypt.compare(inputPassword, user.hashedPassword)
  .then(isValid => {
    if (isValid) { /* login */ }
  });
```

### 5. Generate Security Report

Create comprehensive report:

```markdown
# Security Scan Report
Date: [timestamp]
Project: [name]

## Executive Summary
- 🔴 Critical: [N] issues
- 🟠 High: [N] issues
- 🟡 Medium: [N] issues
- 🟢 Low: [N] issues
- ✅ Passed: [N] checks

## Risk Level: [CRITICAL/HIGH/MEDIUM/LOW]

---

## Critical Issues (Fix Immediately)

### 1. Hardcoded API Key in Production Code
**Severity:** CRITICAL
**File:** src/config/api.ts:15
**Impact:** Exposed credentials could allow unauthorized access
**Fix:**
```typescript
// Before
const API_KEY = "sk_live_12345";

// After
const API_KEY = process.env.API_KEY;
```

### 2. SQL Injection Vulnerability
**Severity:** CRITICAL
**File:** src/db/queries.ts:42
**Impact:** Attackers could access or modify database
**Fix:**
```typescript
// Use parameterized queries
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
```

---

## High Priority Issues

### 1. Vulnerable Dependency: express@4.16.0
**CVE:** CVE-2024-12345
**Severity:** HIGH
**Description:** Denial of Service vulnerability
**Fix:** `npm install express@4.18.0`

---

## Medium Priority Issues

### 1. Missing Security Headers
**Files Affected:** All routes
**Impact:** Reduced protection against common attacks
**Fix:**
```javascript
app.use(helmet());
```

---

## Low Priority Issues

### 1. Outdated Dependency
**Package:** lodash@4.17.15
**Current:** 4.17.15
**Latest:** 4.17.21
**Fix:** `npm update lodash`

---

## Passed Checks ✅
- No exposed environment variables
- Password hashing implemented correctly
- HTTPS enforced
- Rate limiting configured
- Input validation present

---

## Recommendations

### Immediate Actions (Today)
- [ ] Remove hardcoded API keys
- [ ] Fix SQL injection vulnerability
- [ ] Update vulnerable dependencies

### Short Term (This Week)
- [ ] Add security headers
- [ ] Implement CSRF protection
- [ ] Add rate limiting to auth endpoints

### Long Term (This Month)
- [ ] Security training for team
- [ ] Regular automated scans
- [ ] Penetration testing
- [ ] Security policy documentation

---

## Remediation Guide

### For Developers
1. Fix critical issues first
2. Update dependencies
3. Add security tests
4. Review with security team

### For DevOps
1. Update environment variables
2. Configure security headers
3. Set up automated scanning
4. Monitor for new vulnerabilities
```

### 6. Automated Remediation

For each fixable issue, provide automated fix:

```markdown
## Auto-Fix Available

### Issue: Outdated Dependencies
**Fix:**
```bash
npm update
npm audit fix
npm audit fix --force  # for breaking changes
```

### Issue: Missing Security Headers
**Fix:**
```bash
npm install helmet
```

```javascript
// Add to server configuration
const helmet = require('helmet');
app.use(helmet());
```

### Issue: Weak Password Hashing
**Fix:**
```bash
npm install bcrypt
```

```javascript
// Update authentication
const bcrypt = require('bcrypt');
const hashedPassword = await bcrypt.hash(password, 10);
```
```

### 7. Create Security Tasks

Generate actionable tasks:

```markdown
## Security Action Items

### Critical (Do Now)
- [ ] Remove API key from src/config/api.ts
- [ ] Fix SQL injection in src/db/queries.ts
- [ ] Update express to 4.18.0

### High (This Week)
- [ ] Add helmet middleware
- [ ] Implement rate limiting
- [ ] Update all high-severity dependencies

### Medium (This Month)
- [ ] Add CSRF protection
- [ ] Implement security logging
- [ ] Create security documentation

### Low (Backlog)
- [ ] Update low-severity dependencies
- [ ] Improve error messages (don't leak info)
- [ ] Add security tests
```

## Security Scanning Tools

### Node.js Ecosystem
```bash
# npm audit
npm audit

# Snyk
npx snyk test

# ESLint security plugin
npm install --save-dev eslint-plugin-security
```

### Python Ecosystem
```bash
# Safety
safety check

# Bandit
bandit -r src/

# pip-audit
pip-audit
```

### General Tools
```bash
# Trivy (containers, dependencies)
trivy fs .

# GitLeaks (secrets)
gitleaks detect --source . --verbose

# SonarQube
sonar-scanner
```

## Best Practices

✅ **Do:**
- Scan before every deployment
- Fix critical issues immediately
- Keep dependencies updated
- Use environment variables for secrets
- Implement least privilege access
- Log security events
- Regular security reviews

❌ **Don't:**
- Ignore security warnings
- Hardcode sensitive data
- Use deprecated packages
- Trust user input
- Disable security features
- Skip security testing
- Delay security updates

## Customization

### Scan Specific Areas

```bash
/security-scan --focus=dependencies
/security-scan --focus=code
/security-scan --focus=config
```

### Set Severity Threshold

```bash
/security-scan --min-severity=high
```

### Export Report

```bash
/security-scan --output=security-report.json
```

## Integration

### CI/CD Integration

```yaml
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Security Scan
        run: npm audit
```

### Pre-commit Hook

```bash
# .git/hooks/pre-commit
#!/bin/sh
npm audit --audit-level=high
```

## Success Criteria

- [ ] All critical vulnerabilities fixed
- [ ] High-priority issues addressed
- [ ] Dependencies updated
- [ ] Security configurations reviewed
- [ ] Report generated
- [ ] Tasks created for remaining issues

## Related Workflows

- `/dependency-audit` - Deep dependency analysis
- `/code-review-assistant` - Include security review
- `/lint-and-format` - Code quality checks
- `/run-tests-and-fix` - Security tests

---

## 🔗 Related Arsenal Items

**💭 Memories:**
- [FastAPI Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/fastapi-memory.md) - Security best practices for APIs
- [Next.js Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/nextjs-app-router-memory.md) - Frontend security patterns

**📝 Prompts:**
- [Security Agent](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/custom-agents/specialized-agents/security-agent.md) - Specialized security audit agent

**🔗 Examples:**
- [API Service](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/api-service) - Security workflow for APIs

---

**This workflow protects your codebase from security vulnerabilities!**
