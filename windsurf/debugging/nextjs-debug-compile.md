---
id: wkf.nextjs-debug-compile
type: workflow
title: Next.js Compilation Error Debug Workflow
tags: [debugging, nextjs, troubleshooting, runbook, error-resolution]
summary: Systematic 4-phase workflow for debugging Next.js compilation errors - from identifying error types to preventing recurrence.
version: 1
---

# Next.js Compilation Error Debug Workflow

**Systematic approach to fixing Next.js build and compilation errors.**

---

## 🎯 When to Use This Workflow

Use this when you encounter:
- ❌ Build failures (`npm run build` fails)
- ❌ Compilation errors in dev mode
- ❌ TypeScript errors blocking hot reload
- ❌ "Module not found" errors
- ❌ Hydration mismatches
- ❌ Runtime crashes in browser

**Success Rate:** 90%+ when followed systematically

---

## 📋 Phase 1: Identify Error Type

### Step 1.1: Read the Error Message

**Location:** Terminal output or browser console

**Common error patterns:**

#### Type 1: Duplicate Definitions
```
Error: Identifier 'setInputValue' has already been declared
```

**Cause:** Variable/function defined multiple times  
**Next Step:** Go to Phase 2.1

#### Type 2: Type Mismatches
```
Error: Property 'onSend' does not exist on type 'UseChatHelpers'
```

**Cause:** TypeScript type incompatibility  
**Next Step:** Go to Phase 2.2

#### Type 3: Object Rendering
```
Error: Objects are not valid as a React child (found: [object Error])
```

**Cause:** Trying to render Error object directly  
**Next Step:** Go to Phase 2.3

#### Type 4: Module Not Found
```
Error: Module not found: Can't resolve 'some-package'
```

**Cause:** Missing dependency or wrong import path  
**Next Step:** Go to Phase 2.4

#### Type 5: Hydration Mismatch
```
Warning: Text content did not match. Server: "..." Client: "..."
```

**Cause:** Server and client render differently  
**Next Step:** Go to Phase 2.5

#### Type 6: Invalid JSX
```
Error: Adjacent JSX elements must be wrapped in an enclosing tag
```

**Cause:** Syntax error in JSX  
**Next Step:** Go to Phase 2.6

---

## 🔍 Phase 2: Locate and Analyze

### Step 2.1: Duplicate Definitions

**Find the duplicates:**

```bash
# Search for the duplicate identifier
grep -n "setInputValue" src/**/*.tsx
# or in PowerShell:
Select-String -Path "src/**/*.tsx" -Pattern "setInputValue"
```

**Common patterns:**

```typescript
// ❌ BAD - Defined twice
const [inputValue, setInputValue] = useState('');
const setInputValue = (value: string) => { /* ... */ };

// ✅ GOOD - Define once
const [inputValue, setInputValue] = useState('');
// Use setInputValue from useState
```

**Fix:** Remove one of the definitions (usually the redundant one)

---

### Step 2.2: Type Mismatches

**Understand the type error:**

```typescript
// Error: Property 'onSend' does not exist on type 'UseChatHelpers'
```

**Steps:**
1. Check the actual type definition (Ctrl+Click in VSCode)
2. Compare with your usage
3. Check library version (breaking changes?)

**Common causes:**

#### Cause A: Library API Changed
```typescript
// Old API (v1)
const { onSend } = useChat();

// New API (v2) - property renamed
const { handleSubmit } = useChat();
```

**Fix:** Update to new API

#### Cause B: Wrong Type Import
```typescript
// ❌ Wrong import
import { UseChatHelpers } from '@ai-sdk/old-package';

// ✅ Correct import
import { UseChatHelpers } from '@ai-sdk/react';
```

**Fix:** Import from correct package

#### Cause C: Type Doesn't Match Reality
```typescript
// Runtime works but TypeScript complains
const helpers = useChat();
helpers.onSend(message); // Type error but works

// ✅ Temporary fix with type assertion + comment
const helpers = useChat() as any; // TODO: Update types when library updates
helpers.onSend(message);
```

**Fix:** Use type assertion with TODO comment

---

### Step 2.3: Object Rendering Errors

**Find the problematic render:**

```typescript
// ❌ BAD - Crashes
{error && <div>{error}</div>}

// ✅ GOOD - Safe
{error && <div>{error.message}</div>}
// or
{error && <div>{error instanceof Error ? error.message : String(error)}</div>}
```

**See:** [React Error Rendering Rule](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/by-framework/react-error-handling.md)

---

### Step 2.4: Module Not Found

**Diagnostic steps:**

```bash
# 1. Check if package is installed
npm list some-package

# 2. Check package.json
cat package.json | grep some-package

# 3. Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

**Common causes:**

#### Cause A: Package Not Installed
```bash
# Fix: Install it
npm install some-package
```

#### Cause B: Wrong Import Path
```typescript
// ❌ BAD
import { Component } from '@/components/Component';
// File is actually at src/component/Component.tsx

// ✅ GOOD
import { Component } from '@/component/Component';
```

#### Cause C: Case Sensitivity
```typescript
// ❌ BAD - Wrong case
import { Button } from '@/components/button';
// File is Button.tsx

// ✅ GOOD
import { Button } from '@/components/Button';
```

**Fix:** Correct the import path

---

### Step 2.5: Hydration Mismatches

**Find the mismatch:**

Look for:
- Date/time rendering (different on server vs client)
- Random values (Math.random())
- Browser-only APIs (window, localStorage)
- Conditional rendering based on client state

**Common patterns:**

```typescript
// ❌ BAD - Hydration mismatch
function Component() {
  return <div>{new Date().toLocaleString()}</div>;
}

// ✅ GOOD - Use useEffect for client-only
function Component() {
  const [time, setTime] = useState('');
  
  useEffect(() => {
    setTime(new Date().toLocaleString());
  }, []);
  
  return <div>{time || 'Loading...'}</div>;
}
```

```typescript
// ❌ BAD - Browser API on server
function Component() {
  const theme = localStorage.getItem('theme');
  return <div className={theme}>Content</div>;
}

// ✅ GOOD - Use useEffect
function Component() {
  const [theme, setTheme] = useState('light');
  
  useEffect(() => {
    setTheme(localStorage.getItem('theme') || 'light');
  }, []);
  
  return <div className={theme}>Content</div>;
}
```

---

### Step 2.6: Invalid JSX

**Find the syntax error:**

```typescript
// ❌ BAD - Multiple root elements
function Component() {
  return (
    <div>First</div>
    <div>Second</div>
  );
}

// ✅ GOOD - Wrapped in fragment
function Component() {
  return (
    <>
      <div>First</div>
      <div>Second</div>
    </>
  );
}
```

**Common JSX errors:**
- Missing closing tags
- Self-closing tags not closed (`<img>` should be `<img />`)
- JavaScript expressions not wrapped in `{}`
- Conditional rendering syntax errors

---

## 🔧 Phase 3: Apply Fix

### Step 3.1: Make Targeted Changes

**Principles:**
- ✅ Change one thing at a time
- ✅ Test after each change
- ✅ Keep changes minimal
- ✅ Add comments explaining temporary fixes

### Step 3.2: Common Fix Patterns

#### Fix Pattern 1: Remove Duplicates
```typescript
// Before
const [value, setValue] = useState('');
const setValue = (v: string) => setLocalValue(v);

// After
const [value, setValue] = useState('');
// Removed duplicate setValue - use the one from useState
```

#### Fix Pattern 2: Type Assertions
```typescript
// Before
const helpers = useChat();
helpers.newFeature(); // Type error

// After
const helpers = useChat() as any; // TODO: Update @ai-sdk/react types
helpers.newFeature(); // Works in runtime
```

#### Fix Pattern 3: Error Message Conversion
```typescript
// Before
{error && <div>{error}</div>}

// After
{error && <div>{error.message || String(error)}</div>}
```

#### Fix Pattern 4: Install Missing Packages
```bash
# Check what's needed
npm install

# Or install specific package
npm install missing-package
```

#### Fix Pattern 5: Clear Cache
```bash
# Next.js cache issues
rm -rf .next
npm run dev

# Complete clean
rm -rf .next node_modules package-lock.json
npm install
npm run dev
```

---

## ✅ Phase 4: Verify and Prevent

### Step 4.1: Verify the Fix

**Checklist:**
- [ ] Build succeeds: `npm run build`
- [ ] Dev server starts: `npm run dev`
- [ ] Hot reload works (edit file, see changes)
- [ ] No new errors in terminal
- [ ] No errors in browser console
- [ ] Application functions correctly
- [ ] All features still work

### Step 4.2: Test Edge Cases

```typescript
// Test error states
setError(new Error('Test error'));

// Test loading states
setLoading(true);

// Test with different data
// Test with null/undefined
// Test with empty arrays/objects
```

### Step 4.3: Document the Fix

**Add comments:**
```typescript
// Fixed: Duplicate setInputValue definition (removed redundant one)
const [inputValue, setInputValue] = useState('');

// HACK: Type assertion needed until @ai-sdk/react updates types
// Remove when library reaches v3.0
const helpers = useChat() as any;
```

**Update CHANGELOG or commit message:**
```
fix: remove duplicate setInputValue definition

- Removed redundant setInputValue function
- Keep useState version which is used throughout component
- Fixes compilation error: "Identifier already declared"
```

### Step 4.4: Prevent Recurrence

**Add linting rules:**
```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'no-duplicate-declarations': 'error',
    'react/jsx-no-leaked-render': 'error'
  }
};
```

**Add pre-commit hooks:**
```bash
# .husky/pre-commit
npm run type-check
npm run lint
```

**Update team documentation:**
```markdown
## Common Errors

### Duplicate Declarations
- Always check if variable already exists from useState
- Search file for identifier before creating new one

### Error Rendering
- Never render Error objects directly: {error} ❌
- Always use error.message or String(error) ✓
```

---

## 🚨 Emergency Procedures

### When Build is Completely Broken

```bash
# Step 1: Stash your changes
git stash

# Step 2: Verify main branch works
git checkout main
npm install
npm run build

# Step 3: If main works, your changes broke it
git checkout your-branch
git stash pop

# Step 4: Bisect to find breaking change
# Undo changes one by one until it works

# Step 5: Re-apply changes carefully
# Test after each change
```

### When Dev Server Won't Start

```bash
# Nuclear option - complete rebuild
rm -rf .next node_modules package-lock.json
npm install
npm run dev

# If still fails, check:
# - Port 3000 in use? (try different port)
# - Node version correct? (need 18+)
# - Environment variables set?
```

### When TypeScript is Confused

```bash
# Restart TypeScript server in VSCode
# Cmd+Shift+P (Mac) or Ctrl+Shift+P (Windows)
# "TypeScript: Restart TS Server"

# Or restart VSCode completely
```

---

## 📊 Error Decision Tree

```
Compilation Error?
│
├─ Type Error?
│  ├─ Property doesn't exist
│  │  ├─ Check library docs for API changes
│  │  ├─ Verify import source
│  │  └─ Use type assertion if runtime works
│  │
│  ├─ Cannot find name
│  │  ├─ Check import statements
│  │  ├─ Check spelling/case
│  │  └─ Verify file exports
│  │
│  └─ Type not assignable
│     ├─ Check type definitions
│     ├─ Use type casting if safe
│     └─ Fix type incompatibility
│
├─ Syntax Error?
│  ├─ Missing bracket/paren
│  │  └─ Check line above error
│  │
│  ├─ Unexpected token
│  │  └─ Check for typos/syntax
│  │
│  └─ Invalid JSX
│     ├─ Wrap multiple elements
│     └─ Check closing tags
│
├─ Module Error?
│  ├─ Cannot find module
│  │  ├─ Run npm install
│  │  ├─ Check import path
│  │  └─ Check file exists
│  │
│  ├─ Module parse failed
│  │  └─ Check webpack/next config
│  │
│  └─ Version conflict
│     └─ Resolve in package.json
│
└─ Runtime Error?
   ├─ Hydration mismatch
   │  ├─ Use useEffect for client-only code
   │  └─ Avoid Date/random on server
   │
   ├─ Objects not valid (Error rendering)
   │  └─ Convert to error.message
   │
   └─ Infinite loop
      └─ Check useEffect dependencies
```

---

## 💡 Pro Tips

### 1. Read Error Messages Carefully
Don't skim - the message tells you exactly what's wrong.

### 2. Check Recent Changes First
99% of errors come from code you just wrote.

### 3. Use Git to Isolate
```bash
git diff  # See what changed
git checkout -- file.tsx  # Undo changes to one file
```

### 4. Restart Development Server
Sometimes hot reload gets confused. Restart fixes it.

### 5. Clear All Caches
```bash
rm -rf .next node_modules package-lock.json
npm install
```

### 6. Check Dependencies
```bash
npm outdated  # See what's outdated
npm list package-name  # Check if installed
```

### 7. Verify Node Version
```bash
node --version  # Should be 18+
nvm use 20  # Switch to Node 20
```

### 8. Use TypeScript Language Server
VSCode shows type errors inline - fix before runtime.

### 9. Enable Strict Mode
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true
  }
}
```

### 10. Keep Browser Console Open
Many errors show more detail in browser console.

---

## 🔗 Related Arsenal Items

**⚙️ Rule:**
- [React Error Rendering](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/by-framework/react-error-handling.md) - Prevents Error object crashes

**💭 Memory:**
- [Next.js App Router Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/nextjs-app-router-memory.md) - Next.js patterns and conventions
- [BrowserTools MCP](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/development-tools/browsertools-mcp.md) - Access browser console from Windsurf

**🤖 Script:**
- [Next.js Dev Starter](https://github.com/ChrisTansey007/ai-scripts-arsenal/tree/main/scripts/development/nextjs-start-dev) - Smart dev server startup

**🔗 Example:**
- [Next.js Debugging Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/nextjs-debugging) - Complete debugging setup

---

## 📚 Common Error Reference

### Most Frequent Next.js Errors

1. **"Objects are not valid as a React child"** → Use error.message
2. **"Cannot find module"** → npm install or fix path
3. **"Property does not exist on type"** → Check types/API changes
4. **"Hydration mismatch"** → Use useEffect for client code
5. **"Already declared"** → Remove duplicate definition
6. **"Adjacent JSX elements"** → Wrap in fragment
7. **"Cannot read property of undefined"** → Add optional chaining
8. **"Maximum update depth exceeded"** → Fix useEffect dependencies
9. **"Failed to compile"** → Check syntax errors
10. **"Port 3000 is already in use"** → Kill process or use different port

---

## ✅ Success Checklist

After fixing error:
- [ ] Build succeeds
- [ ] Dev server runs
- [ ] Hot reload works
- [ ] No console errors
- [ ] Feature works correctly
- [ ] Tests pass
- [ ] Added comment explaining fix
- [ ] Committed with clear message
- [ ] Documented if needed

---

**Result: Systematic approach turns frustrating errors into quick fixes!** 🎯
