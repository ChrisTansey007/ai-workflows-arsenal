---
id: wkf.local-test-deploy
type: workflow
title: Test Locally Before Deploying to Production
tags: [runbook, testing, deployment, local-first, quality-assurance]
summary: Comprehensive 4-phase workflow for testing applications locally against production/staging databases before deploying - prevents production failures and 404 errors.
version: 1
---

# Test Locally Before Deploying to Production

**Enforce local testing against real data before pushing to production - catch issues in minutes, not hours.**

---

## 🎯 Why This Workflow

**The Problem:**
- Deploy code to production
- Users report 404 errors or broken features
- Realize endpoints were commented out or broken
- Emergency rollback required
- Lost time: 2-4 hours

**The Solution:**
- Test locally against staging/production database
- Catch issues before deployment
- Fix in development environment
- Deploy with confidence
- Time saved: 2-4 hours per incident

**Success Rate:** 95%+ of production issues caught before deployment

---

## 📋 Phase 1: Setup Local Environment

### Step 1.1: Install Dependencies

```bash
# Ensure all dependencies installed
npm install  # or pip install -r requirements.txt

# Verify versions match production
node --version
python --version
```

### Step 1.2: Configure Environment Variables

**Create `.env.local` file:**

```bash
# .env.local - Local development with staging database

# Database (connect to staging, NOT production)
DATABASE_URL=postgresql+psycopg://user:pass@staging-db.example.com:5432/myapp

# API Keys (use test/staging keys)
API_KEY=test_key_123
STRIPE_KEY=sk_test_123

# Service URLs (point to staging)
EXTERNAL_API_URL=https://staging-api.example.com

# Debug settings
DEBUG=true
LOG_LEVEL=debug

# CORS (allow localhost)
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173
```

**Important:**
- ✅ Use staging database, NOT production
- ✅ Use test API keys
- ✅ Enable debug logging
- ✅ Allow localhost CORS

### Step 1.3: Verify Database Connection

**Test connection:**

```bash
# Python/FastAPI
python -c "from app.db.session import engine; import asyncio; asyncio.run(engine.connect())"

# Node.js/Prisma
npx prisma db pull

# Check you can query data
curl http://localhost:8000/health
```

**Expected:** Connection successful, can read staging data

---

## Phase 2: Start Local Services

### Step 2.1: Start Backend

**Python/FastAPI:**
```bash
# Start with staging database connection
DATABASE_URL="postgresql+psycopg://..." uvicorn app.main:app --reload --port 8000

# Or use script
./start-backend-local.sh
```

**Node.js/Express:**
```bash
# Start with environment variables
NODE_ENV=development DATABASE_URL="..." npm run dev
```

**Verify backend started:**
```bash
curl http://localhost:8000/health
# Expected: {"status": "ok"}
```

### Step 2.2: Start Frontend

**React/Vite:**
```bash
# Configure to use local backend
VITE_API_URL=http://localhost:8000 npm run dev
```

**Next.js:**
```bash
NEXT_PUBLIC_API_URL=http://localhost:8000 npm run dev
```

**Verify frontend started:**
- Open http://localhost:5173 (or 3000)
- Should load without errors

### Step 2.3: Check Logs

**Backend logs should show:**
```
INFO: Started server process
INFO: Waiting for application startup
INFO: Application startup complete
INFO: Uvicorn running on http://0.0.0.0:8000
```

**Frontend logs should show:**
```
VITE ready in 324 ms
➜  Local:   http://localhost:5173/
```

---

## Phase 3: Comprehensive Testing

### Step 3.1: Test Critical API Endpoints

**Create test script:**

```bash
#!/bin/bash
# test-endpoints.sh

BASE_URL="http://localhost:8000/api/v1"

echo "Testing critical endpoints..."

# Health check
echo "→ Health check"
curl -s $BASE_URL/../health | jq

# List endpoints
echo "→ GET /companies"
curl -s $BASE_URL/companies | jq '. | length'

echo "→ GET /customers"
curl -s $BASE_URL/customers | jq '. | length'

echo "→ GET /projects"
curl -s $BASE_URL/projects | jq '. | length'

# Get by ID
echo "→ GET /companies/1"
curl -s $BASE_URL/companies/1 | jq '.name'

# Test 404 handling
echo "→ GET /companies/99999 (should 404)"
curl -s -w "%{http_code}" $BASE_URL/companies/99999

echo "✓ All endpoint tests complete"
```

**Run tests:**
```bash
chmod +x test-endpoints.sh
./test-endpoints.sh
```

**Expected results:**
- ✅ All endpoints return 200 (not 404)
- ✅ Data is returned (not empty)
- ✅ 404 endpoints properly return 404
- ✅ No 500 errors

### Step 3.2: Test Frontend Integration

**Open browser DevTools (F12):**

1. **Network Tab:**
   - Navigate to main pages
   - Check all API calls
   - Should see 200 responses, not 404

2. **Console Tab:**
   - Should be no errors
   - No "Failed to fetch" messages
   - No CORS errors

3. **Application Tab:**
   - Check local storage
   - Verify data is stored correctly

**Test user flows:**
```
1. Load homepage
   → Check: Companies list displays
   → Check: No loading errors

2. Click on company
   → Check: Company details load
   → Check: Related data displays

3. Navigate to customers page
   → Check: Customer list displays
   → Check: Pagination works

4. Test search/filter
   → Check: Results update correctly
   → Check: Empty states handled

5. Test forms (if applicable)
   → Check: Validation works
   → Check: Submit succeeds
   → Check: Success message shows
```

### Step 3.3: Test Error Scenarios

**Simulate errors:**

```bash
# Test with invalid data
curl -X POST http://localhost:8000/api/v1/companies \
  -H "Content-Type: application/json" \
  -d '{"name": ""}'  # Empty name should fail

# Test non-existent resource
curl http://localhost:8000/api/v1/companies/99999

# Test missing required fields
curl -X POST http://localhost:8000/api/v1/companies \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Expected:**
- ✅ Proper error messages
- ✅ Appropriate status codes (400, 404, 422)
- ✅ No 500 errors (unless expected)

### Step 3.4: Performance Check

**Monitor response times:**

```bash
# Check slow endpoints
time curl http://localhost:8000/api/v1/companies

# Should complete in < 1 second for simple queries
```

**Check database queries:**
```bash
# Enable query logging in backend
LOG_LEVEL=debug uvicorn app.main:app

# Watch for N+1 queries or slow queries
```

### Step 3.5: Cross-Browser Testing (if critical)

**Test in multiple browsers:**
- Chrome/Edge
- Firefox
- Safari (if on Mac)

**Look for:**
- Layout issues
- JavaScript errors
- API call failures

---

## Phase 4: Deploy with Confidence

### Step 4.1: Final Checklist

**Before deploying, verify:**
- [ ] All critical endpoints tested and working
- [ ] Frontend loads and displays data correctly
- [ ] No 404 errors in network tab
- [ ] No console errors
- [ ] Error handling works properly
- [ ] Forms submit successfully (if applicable)
- [ ] Search/filter works (if applicable)
- [ ] Pagination works (if applicable)
- [ ] Mobile responsive (if applicable)
- [ ] Performance acceptable (< 2 sec load)

### Step 4.2: Document Test Results

**Create test summary:**

```markdown
# Pre-Deployment Test Results
Date: 2025-10-19
Tester: [Your Name]
Branch: feature/new-api-endpoints

## Environment
- Backend: Local (http://localhost:8000)
- Database: Staging (db-staging.example.com)
- Frontend: Local (http://localhost:5173)

## Test Results

### API Endpoints (✓ All Passing)
- ✓ GET /health - 200 OK
- ✓ GET /api/v1/companies - 200 OK (12 results)
- ✓ GET /api/v1/customers - 200 OK (45 results)
- ✓ GET /api/v1/companies/1 - 200 OK
- ✓ GET /api/v1/companies/99999 - 404 Not Found (expected)

### Frontend (✓ All Passing)
- ✓ Homepage loads correctly
- ✓ Companies list displays with data
- ✓ Company detail page works
- ✓ No console errors
- ✓ No network errors (404s)

### Performance
- Homepage: 543ms
- API /companies: 127ms
- API /customers: 89ms

## Issues Found
None - ready to deploy

## Deployment Approval
✓ Approved for deployment to staging/production
```

### Step 4.3: Commit and Push

```bash
# Stage changes
git add .

# Commit with clear message
git commit -m "feat: add companies and customers API endpoints

- Implement GET /companies endpoint with pagination
- Implement GET /customers endpoint with filtering
- Add error handling for 404 cases
- Tested locally against staging database
- All endpoints return expected data"

# Push to branch
git push origin feature/new-api-endpoints
```

### Step 4.4: Deploy

**Deploy to staging first:**
```bash
# Create PR
gh pr create --title "Add companies/customers endpoints" --body "Tested locally ✓"

# After approval, merge
git checkout main
git pull
git merge feature/new-api-endpoints
git push origin main

# Deploy triggers automatically or manually
```

**Monitor deployment:**
```bash
# Watch logs
heroku logs --tail  # or your deployment platform

# Check health
curl https://staging.example.com/health

# Test endpoints in staging
curl https://staging.example.com/api/v1/companies | jq
```

### Step 4.5: Verify Production

**After deployment:**

```bash
# Test production endpoints
curl https://api.example.com/health
curl https://api.example.com/api/v1/companies | jq '. | length'

# Monitor for errors
# Check error tracking (Sentry, Rollbar, etc.)
```

**If issues found:**
- Check logs immediately
- Compare with local testing
- Rollback if critical

---

## 🚨 Rollback Procedure

### When to Rollback

**Rollback immediately if:**
- 500 errors on critical endpoints
- Data corruption detected
- Security vulnerability discovered
- Major feature completely broken

**Consider fix-forward if:**
- Minor UI issues
- Non-critical feature broken
- Easy 5-minute fix available

### How to Rollback

**Git revert:**
```bash
# Find the bad commit
git log --oneline -10

# Revert it
git revert abc123

# Push
git push origin main
```

**Platform-specific rollback:**
```bash
# Heroku
heroku rollback

# Vercel
vercel rollback

# Manual deployment
git checkout main
git reset --hard HEAD~1
git push --force origin main
./deploy.sh
```

**Notify team:**
```
🚨 Rollback performed
Reason: [Brief description]
Reverted: [Commit hash]
Status: Investigating issue
ETA: [Time estimate]
```

---

## 💡 Pro Tips

### 1. Create Helper Scripts

**start-local-test.sh:**
```bash
#!/bin/bash
set -e

echo "🚀 Starting local test environment..."

# Load environment
export $(cat .env.local | xargs)

# Start backend in background
uvicorn app.main:app --reload --port 8000 &
BACKEND_PID=$!

# Wait for backend
sleep 3

# Start frontend
cd frontend
npm run dev &
FRONTEND_PID=$!

echo "✓ Backend running on http://localhost:8000"
echo "✓ Frontend running on http://localhost:5173"
echo ""
echo "Press Ctrl+C to stop"

# Cleanup on exit
trap "kill $BACKEND_PID $FRONTEND_PID" EXIT
wait
```

### 2. Use Database Snapshots

```bash
# Take snapshot before testing
pg_dump staging_db > backup-$(date +%Y%m%d).sql

# Restore if needed
psql staging_db < backup-20251019.sql
```

### 3. Automate Testing

**package.json:**
```json
{
  "scripts": {
    "test:local": "bash scripts/test-endpoints.sh",
    "test:integration": "jest --config jest.integration.js",
    "pre-deploy": "npm run test:local && npm run test:integration"
  }
}
```

### 4. Document Common Issues

**docs/local-testing-issues.md:**
```markdown
# Common Local Testing Issues

## Backend won't start
- Check DATABASE_URL is correct
- Verify port 8000 is not in use: `lsof -i :8000`
- Check Python version: `python --version`

## Database connection fails
- Check VPN is connected (if required)
- Verify credentials in .env.local
- Test with: `psql $DATABASE_URL`

## Frontend shows CORS errors
- Backend must allow localhost
- Check ALLOWED_ORIGINS in backend config
```

### 5. Use Feature Flags

```python
# Instead of commenting out code
if FEATURE_FLAGS.get("new_api_enabled"):
    app.include_router(new_api.router)
```

### 6. Test with Real Data Patterns

```bash
# Get real data patterns from production
psql production_db -c "COPY (SELECT * FROM companies LIMIT 10) TO STDOUT" > test-data.csv

# Load into staging
psql staging_db -c "COPY companies FROM STDIN" < test-data.csv
```

---

## 🔗 Related Arsenal Items

**⚙️ Rules:**
- [Windows Async Event Loop](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/database/windows-async-eventloop.md) - Fix async DB on Windows
- [Raw SQL Fallback](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/database/raw-sql-fallback.md) - Quick API development

**💭 Memory:**
- [FastAPI Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/fastapi-memory.md) - FastAPI patterns

**📝 Prompt:**
- [Implement CRUD with Raw SQL](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/development/api/implement-crud-raw-sql.md) - API development template

**🔗 Example:**
- [Database API Development](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/database-api-development) - Complete example

---

## 📊 Success Metrics

**Track these metrics:**
- Production incidents before workflow: X per month
- Production incidents after workflow: Y per month
- Reduction: (X-Y)/X * 100%

**Expected improvements:**
- 80-90% reduction in production incidents
- 2-4 hours saved per prevented incident
- Increased developer confidence
- Faster deployment cycles

---

## ✅ Checklist Template

**Print and use for each deployment:**

```markdown
# Pre-Deployment Checklist

## Environment Setup
- [ ] Dependencies installed and up to date
- [ ] .env.local configured with staging database
- [ ] Database connection verified

## Local Services
- [ ] Backend started successfully
- [ ] Frontend started successfully
- [ ] Both services running without errors

## API Testing
- [ ] Health endpoint returns 200
- [ ] All CRUD endpoints tested
- [ ] List endpoints return data
- [ ] Detail endpoints return data
- [ ] 404 handling works correctly
- [ ] Error handling works correctly

## Frontend Testing
- [ ] Homepage loads without errors
- [ ] All pages load without 404s
- [ ] Network tab shows all 200 responses
- [ ] Console shows no errors
- [ ] Data displays correctly
- [ ] User interactions work

## Performance
- [ ] API responses < 2 seconds
- [ ] Page load < 3 seconds
- [ ] No slow queries detected

## Documentation
- [ ] Test results documented
- [ ] Issues noted (if any)
- [ ] Commit message includes testing note

## Deployment
- [ ] Code committed with clear message
- [ ] PR created (if applicable)
- [ ] Deployed to staging first
- [ ] Staging verified
- [ ] Production deployment approved
- [ ] Production verified

## Post-Deployment
- [ ] Monitoring checked (no errors)
- [ ] Performance acceptable
- [ ] Team notified
```

---

**Result: Deploy with confidence knowing everything works - catch issues in development, not production!** 🎯
