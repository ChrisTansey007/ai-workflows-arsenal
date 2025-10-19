---
id: wkf.oauth-email-debug
type: workflow
title: OAuth Email Provider Debugging Workflow
tags: [oauth, debugging, email, imap, smtp, troubleshooting, authentication]
summary: Systematic 5-phase debugging workflow for OAuth email integration issues - resolve silent failures, missing popups, CORS errors, and provider-specific problems in 30 minutes.
version: 1
---

# OAuth Email Provider Debugging Workflow

**Systematic approach to debug OAuth email integration issues - from silent failures to successful authentication.**

---

## 🎯 When to Use This Workflow

**Symptoms:**
- OAuth popup doesn't appear
- "No response from server" errors
- CORS errors in browser console
- 404/500 errors on OAuth endpoints
- Authorization succeeds but email sync fails
- Works in development but fails in production

**Time to Debug:** 15-30 minutes (vs 2-4 hours trial and error)

---

## 📋 Phase 1: Quick Verification

### Step 1.1: Check Browser Console

**Open browser DevTools (F12) → Console tab**

**Look for:**
```
✅ GOOD: POST http://localhost:8001/api/v1/oauth/google/authorize 200
❌ BAD:  POST http://localhost:8001/api/v1/oauth/google/authorize net::ERR_CONNECTION_REFUSED
❌ BAD:  CORS error: No 'Access-Control-Allow-Origin' header
```

**Quick diagnosis:**
- **Connection refused** → Backend not running
- **CORS error** → Origin not in ALLOWED_ORIGINS
- **404** → Route not registered
- **500** → Check backend logs

### Step 1.2: Verify Backend is Running

```bash
# Check backend process
ps aux | grep uvicorn

# Or check port
lsof -i :8001  # macOS/Linux
netstat -ano | findstr :8001  # Windows

# Expected: Should show running process
```

### Step 1.3: Test Health Endpoint

```bash
# Test backend is responsive
curl http://localhost:8001/health

# Expected response:
# {"status": "ok"}
```

---

## Phase 2: Frontend Debugging

### Step 2.1: Check API Call

**Verify OAuth service is making correct calls:**

```typescript
// In browser console, check network tab
// Look for request to: /api/v1/oauth/PROVIDER/authorize

// Common issues:
❌ Wrong URL: http://localhost:3000/api/... (hitting frontend)
✅ Correct:   http://localhost:8001/api/... (hitting backend)

❌ Wrong method: GET (should be POST for some providers)
✅ Correct:     POST

❌ Missing headers: No Content-Type
✅ Correct:         Content-Type: application/json
```

**Check service configuration:**

```typescript
// frontend/src/services/oauthService.ts

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8001';

// Common mistake: hardcoded localhost without port
❌ const API_URL = 'http://localhost';  
✅ const API_URL = 'http://localhost:8001';

// Common mistake: trailing slash
❌ const API_URL = 'http://localhost:8001/';  
✅ const API_URL = 'http://localhost:8001';
```

### Step 2.2: Check Environment Variables

```bash
# frontend/.env.local or frontend/.env
cat frontend/.env.local | grep VITE_API_URL

# Should match backend URL
VITE_API_URL=http://localhost:8001
```

### Step 2.3: Verify Popup Logic

**Check popup blocker:**
```javascript
// Test in console
const popup = window.open('https://google.com', '_blank', 'width=600,height=600');
if (!popup) {
  console.error('Popup blocked! Enable popups for this site.');
}
```

**Check async/await pattern:**
```typescript
// ❌ BAD: Not awaiting the response
const authUrl = oauthService.getAuthUrl('google');
window.open(authUrl); // undefined!

// ✅ GOOD: Properly awaiting
const authUrl = await oauthService.getAuthUrl('google');
window.open(authUrl, '_blank', 'width=600,height=600');
```

---

## Phase 3: Backend Debugging

### Step 3.1: Test OAuth Endpoint Directly

```bash
# Test authorization URL generation
curl -X POST http://localhost:8001/api/v1/oauth/google/authorize \
  -H "Content-Type: application/json" \
  -d '{}'

# Expected response:
{
  "authorization_url": "https://accounts.google.com/o/oauth2/v2/auth?..."
}

# If you get:
# {"detail": "Not Found"} → Route not registered
# {"detail": "Provider not configured"} → Missing credentials
# Connection refused → Backend not running
```

### Step 3.2: Check Route Registration

```python
# backend/app/api/__init__.py or backend/app/main.py

from app.api.routes import oauth

app.include_router(
    oauth.router,
    prefix="/api/v1",
    tags=["oauth"]
)

# Common mistake: Missing router registration
# Common mistake: Wrong prefix (should match frontend calls)
```

### Step 3.3: Verify Environment Variables

```bash
# backend/.env
cat backend/.env | grep -E "(GOOGLE|MICROSOFT|YAHOO)_(CLIENT_ID|CLIENT_SECRET)"

# Expected (example for Google):
GOOGLE_CLIENT_ID=123456789-abcdefg.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-abc123def456

# Common mistakes:
❌ Empty values: GOOGLE_CLIENT_ID=
❌ Placeholder text: GOOGLE_CLIENT_ID=your-client-id-here
❌ Quotes around values: GOOGLE_CLIENT_ID="123..." (remove quotes)
❌ Spaces: GOOGLE_CLIENT_ID = 123... (no spaces around =)
```

**Load and verify in Python:**
```python
# backend/app/config.py or test script
import os
from dotenv import load_dotenv

load_dotenv()

print(f"Google Client ID: {os.getenv('GOOGLE_CLIENT_ID')}")
print(f"Google Client Secret: {os.getenv('GOOGLE_CLIENT_SECRET')}")

# Should print actual values, not None
```

### Step 3.4: Check Provider Configuration

```python
# backend/app/services/oauth_service.py

PROVIDER_CONFIGS = {
    'google': {
        'client_id': os.getenv('GOOGLE_CLIENT_ID'),
        'client_secret': os.getenv('GOOGLE_CLIENT_SECRET'),
        'authorization_base_url': 'https://accounts.google.com/o/oauth2/v2/auth',
        'token_url': 'https://oauth2.googleapis.com/token',
        'scope': ['https://www.googleapis.com/auth/gmail.readonly'],
        'redirect_uri': f'{os.getenv("BACKEND_URL")}/api/v1/oauth/google/callback'
    }
}

# Common mistakes:
❌ Wrong redirect_uri (doesn't match OAuth console)
❌ Missing/wrong scopes
❌ Typo in URLs
```

---

## Phase 4: CORS & Network Issues

### Step 4.1: Check CORS Configuration

```python
# backend/app/main.py

from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",  # Vite dev server
        "http://localhost:3000",  # React/Next.js dev server
        "https://your-production-domain.com"
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Common mistakes:
❌ Missing frontend URL
❌ Trailing slash mismatch: "http://localhost:5173/" vs "http://localhost:5173"
❌ Wrong port number
❌ Wildcard in production: allow_origins=["*"] (security risk)
```

### Step 4.2: Test CORS Manually

```bash
# Test CORS preflight
curl -X OPTIONS http://localhost:8001/api/v1/oauth/google/authorize \
  -H "Origin: http://localhost:5173" \
  -H "Access-Control-Request-Method: POST" \
  -v

# Look for:
# Access-Control-Allow-Origin: http://localhost:5173
# Access-Control-Allow-Methods: POST
```

### Step 4.3: Check Network/Proxy Issues

```bash
# Test direct connection
curl http://localhost:8001/api/v1/oauth/google/authorize

# If using proxy:
# Disable proxy temporarily
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy

# Or configure proxy exception
export NO_PROXY=localhost,127.0.0.1
```

---

## Phase 5: Provider-Specific Issues

### Google OAuth

**Common Issues:**

**1. "redirect_uri_mismatch" error**
```
❌ Configured: http://localhost:8001/api/v1/oauth/google/callback
✅ Required:   Must match EXACTLY in Google Console

Fix:
1. Go to https://console.cloud.google.com/
2. APIs & Services → Credentials
3. Click your OAuth client
4. Add: http://localhost:8001/api/v1/oauth/google/callback
5. Add: https://your-domain.com/api/v1/oauth/google/callback
```

**2. "access_denied" - Insufficient permissions**
```
Required scopes for Gmail:
- https://www.googleapis.com/auth/gmail.readonly (read emails)
- https://www.googleapis.com/auth/gmail.send (send emails)
- https://www.googleapis.com/auth/gmail.modify (modify emails)

Enable in Google Console:
1. APIs & Services → Library
2. Search "Gmail API"
3. Click Enable
```

**3. "invalid_client" error**
```
Cause: Wrong client_id or client_secret

Fix:
1. Regenerate credentials in Google Console
2. Update .env file
3. Restart backend
```

### Microsoft OAuth

**Common Issues:**

**1. "AADSTS50011: redirect_uri mismatch"**
```
Fix:
1. Go to https://portal.azure.com/
2. Azure Active Directory → App registrations
3. Your app → Authentication
4. Add redirect URI: http://localhost:8001/api/v1/oauth/microsoft/callback
5. Choose "Web" platform
```

**2. "AADSTS65001: consent required"**
```
Required API permissions:
- Mail.Read
- Mail.Send
- IMAP.AccessAsUser.All
- SMTP.Send

Add permissions:
1. App registrations → API permissions
2. Add permission → Microsoft Graph
3. Add delegated permissions above
4. Grant admin consent (if tenant admin)
```

**3. Multi-tenant vs Single-tenant**
```
Common mistake: Using wrong authorization URL

Single-tenant:
https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/authorize

Multi-tenant:
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
```

### Yahoo OAuth

**Common Issues:**

**1. "invalid_request: redirect_uri"**
```
Yahoo requires HTTPS in production!

Development:
✅ http://localhost:8001/api/v1/oauth/yahoo/callback

Production:
❌ http://your-domain.com/... (won't work)
✅ https://your-domain.com/api/v1/oauth/yahoo/callback
```

**2. App password confusion**
```
Yahoo users with passkeys cannot use OAuth without app password for IMAP!

Flow:
1. User authorizes via OAuth (gets token)
2. User must ALSO create app-specific password
3. Use app password for IMAP connection, OAuth token for API calls

See: https://account.yahoo.com/security
```

---

## 🔍 Error Catalog

### Frontend Errors

| Error Message | Cause | Solution |
|--------------|-------|----------|
| `Cannot read properties of undefined` | API call returned undefined | Check `await` on async calls |
| `Network request failed` | Backend not running | Start backend: `uvicorn app.main:app` |
| `CORS policy: No 'Access-Control-Allow-Origin'` | CORS not configured | Add frontend URL to ALLOWED_ORIGINS |
| Popup doesn't appear | Popup blocked or async issue | Enable popups, check await pattern |
| `Failed to fetch` | Wrong API URL | Check VITE_API_URL in .env |

### Backend Errors

| Error Message | Cause | Solution |
|--------------|-------|----------|
| `Provider not configured` | Missing credentials | Add CLIENT_ID/SECRET to .env |
| `404 Not Found` | Route not registered | Check router registration |
| `redirect_uri_mismatch` | URI not in OAuth console | Add redirect URI to provider console |
| `invalid_client` | Wrong credentials | Regenerate and update credentials |
| `access_denied` | Missing scopes/permissions | Enable required APIs in provider console |
| `AADSTS50011` (Microsoft) | Redirect URI not registered | Add URI in Azure portal |

### OAuth Provider Errors

| Error Code | Provider | Meaning | Solution |
|-----------|----------|---------|----------|
| `redirect_uri_mismatch` | Google | URI not in allowlist | Add exact URI to Google Console |
| `access_denied` | All | User declined or no permissions | Check scopes, enable APIs |
| `invalid_grant` | All | Token expired/revoked | Re-authenticate user |
| `AADSTS50011` | Microsoft | Redirect URI mismatch | Update Azure app registration |
| `AADSTS65001` | Microsoft | Admin consent required | Grant admin consent for permissions |
| `invalid_request` | Yahoo | Various issues | Check logs for specific error |

---

## 🧪 Testing & Verification

### Step 1: Test Authorization Flow

```bash
# 1. Start backend
cd backend
uvicorn app.main:app --reload --port 8001

# 2. Start frontend
cd frontend
npm run dev

# 3. Open browser
http://localhost:5173

# 4. Test OAuth flow
# Click "Connect Gmail" (or other provider)
# Should:
# ✅ Show popup with provider login
# ✅ Redirect back to app
# ✅ Display success message
```

### Step 2: Test Token Storage

```bash
# Check database for stored tokens
# (Example with PostgreSQL)
psql -d your_database -c "SELECT user_id, provider, expires_at FROM oauth_tokens;"

# Should show:
# user_id | provider | expires_at
# 1       | google   | 2025-10-20 12:00:00
```

### Step 3: Test Email Sync

```bash
# Test IMAP connection with OAuth token
# (This depends on your implementation)

# Check logs for successful connection
tail -f backend/logs/app.log | grep "IMAP"

# Should show:
# INFO: IMAP connection established for user@gmail.com
```

---

## 🚨 Emergency Fixes

### Quick Fix 1: OAuth Popup Not Showing

```bash
# Check these in order:
1. Backend running? → curl http://localhost:8001/health
2. CORS configured? → Check allow_origins in main.py
3. Credentials set? → grep "CLIENT_ID" backend/.env
4. Route registered? → Check app.include_router(oauth.router)
5. Frontend URL correct? → Check VITE_API_URL in frontend/.env
```

### Quick Fix 2: CORS Errors

```python
# backend/app/main.py
# Add this immediately after app = FastAPI()

from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Quick Fix 3: Provider Not Configured

```bash
# backend/.env
# Add these (replace with your actual values):

GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret

# Restart backend
pkill -f uvicorn
uvicorn app.main:app --reload --port 8001
```

---

## 📊 Debugging Checklist

**Print this and check off as you debug:**

### Environment Setup
- [ ] Backend running on correct port
- [ ] Frontend running and accessible
- [ ] .env files exist and loaded
- [ ] All required credentials present

### Frontend
- [ ] Browser console shows no errors
- [ ] API URL correct (includes port)
- [ ] OAuth service properly awaiting responses
- [ ] Popup blockers disabled

### Backend
- [ ] Health endpoint returns 200
- [ ] OAuth routes registered
- [ ] CORS includes frontend URL
- [ ] Environment variables loaded
- [ ] Provider configs have credentials

### OAuth Provider
- [ ] Redirect URI matches exactly
- [ ] Required scopes/permissions enabled
- [ ] APIs enabled (Gmail API, etc.)
- [ ] Credentials not expired/revoked

### Testing
- [ ] Can generate authorization URL
- [ ] Popup opens with provider login
- [ ] Callback endpoint receives code
- [ ] Token exchange succeeds
- [ ] Token stored in database

---

## 💡 Pro Tips

### 1. Use OAuth Playground for Testing

```bash
# Google OAuth Playground
https://developers.google.com/oauthplayground/

# Microsoft OAuth Playground
https://oauthplay.azurewebsites.net/

# Test your scopes and see exact request/response
```

### 2. Enable Debug Logging

```python
# backend/app/services/oauth_service.py

import logging
logging.basicConfig(level=logging.DEBUG)

logger = logging.getLogger(__name__)

# Add debug logs:
logger.debug(f"Authorization URL: {authorization_url}")
logger.debug(f"Redirect URI: {redirect_uri}")
logger.debug(f"Scopes: {scopes}")
```

### 3. Test with curl

```bash
# Test each OAuth endpoint individually

# 1. Authorization URL
curl -X POST http://localhost:8001/api/v1/oauth/google/authorize

# 2. Callback (simulate)
curl "http://localhost:8001/api/v1/oauth/google/callback?code=test_code&state=test_state"

# 3. Token info
curl http://localhost:8001/api/v1/oauth/tokens \
  -H "Authorization: Bearer $USER_TOKEN"
```

### 4. Use Browser DevTools Network Tab

```
Network tab → Filter: XHR
Look for:
- Status codes (200, 404, 500)
- Response time (> 5s = slow)
- Request/response headers
- Request/response payload
```

### 5. Check OAuth Console Logs

Most providers have logs showing OAuth attempts:
- **Google**: Console → APIs & Services → Credentials → Usage
- **Microsoft**: Azure Portal → App registrations → Your app → Authentication → Activity log
- **Yahoo**: Account → Security → Recent activity

---

## 🔗 Related Arsenal Items

**💭 Memory:**
- [Passkey + App Password Bridge](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/development-tools/email-passkey-auth.md)

**📝 Prompts:**
- [Email Provider Preset Generator](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/development/email/generate-provider-preset.md)

**⚙️ Rules:**
- [Email Documentation Standards](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/documentation/email-setup-docs.md)

**🔗 Example:**
- [Email Integration Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/email-integration)

---

**Result: Systematic OAuth debugging - resolve issues in 15-30 minutes instead of hours!** 🎯
