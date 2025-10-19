---
id: wkf.code-review-assistant
type: workflow
title: Code Review Assistant
tags: [code-review, quality, security, best-practices, automation]
summary: AI-powered code review that checks quality, security, performance, and best practices before submitting for human review.
version: 1
---

# Code Review Assistant

**AI-powered code review before submitting for human review**

## Description

This workflow performs a comprehensive code review of your changes, checking for code quality, security issues, performance concerns, and best practices before you submit for human review.

## Use Case

Use this when:
- Ready to create or update a PR
- Want to catch issues before human review
- Need feedback on code quality
- Want to ensure best practices followed

## Prerequisites

- Git repository initialized
- Changes committed or staged
- Know which files/commits to review

## Usage

```bash
/code-review-assistant
```

## Workflow Steps

### 1. Identify Changes to Review

Determine what to review:

**Unstaged changes:**
```bash
git diff
```

**Staged changes:**
```bash
git diff --staged
```

**Last commit:**
```bash
git diff HEAD~1
```

**Between branches:**
```bash
git diff main..feature-branch
```

**Specific files:**
```bash
git diff [file1] [file2]
```

### 2. Analyze Each Changed File

For each modified file, review:

#### a. Code Quality

**Check for:**
- [ ] Code complexity (cyclomatic complexity)
- [ ] Function length (should be concise)
- [ ] DRY violations (repeated code)
- [ ] Clear naming conventions
- [ ] Proper error handling
- [ ] Edge cases covered

**Example checks:**
```markdown
❌ Function too long (>50 lines)
✅ Clear variable names
⚠️  Consider extracting this logic
```

#### b. Security Issues

**Check for:**
- [ ] SQL injection vulnerabilities
- [ ] XSS vulnerabilities
- [ ] Hardcoded secrets or passwords
- [ ] Insecure cryptography
- [ ] Unvalidated input
- [ ] Exposed sensitive data

**Example findings:**
```markdown
🔴 CRITICAL: Hardcoded API key found
🟡 WARNING: User input not sanitized
✅ No security issues detected
```

#### c. Performance Concerns

**Check for:**
- [ ] N+1 queries
- [ ] Inefficient loops
- [ ] Memory leaks
- [ ] Unnecessary computations
- [ ] Missing indexes (database)
- [ ] Blocking operations

**Example findings:**
```markdown
🟡 Loop over large array could be optimized
🟢 Good use of memoization
⚠️  Consider pagination for this query
```

#### d. Best Practices

**Check for:**
- [ ] Follows language conventions
- [ ] Proper documentation
- [ ] Test coverage
- [ ] Error messages are helpful
- [ ] Logging is appropriate
- [ ] Code is readable

**Example findings:**
```markdown
✅ Good JSDoc comments
❌ Missing unit tests
⚠️  Consider adding error context
```

### 3. Generate Review Report

Create a structured report:

```markdown
# Code Review Report

## Summary
- Files reviewed: [N]
- Issues found: [N]
- Critical: [N]
- Warnings: [N]
- Suggestions: [N]

## Critical Issues 🔴
Must fix before merging:

### 1. [File]: [Issue]
**Line:** [number]
**Problem:** [Description]
**Fix:** [Suggested solution]
**Code:**
```language
[problematic code]
```

## Warnings 🟡
Should address:

### 1. [File]: [Issue]
**Line:** [number]
**Problem:** [Description]
**Suggestion:** [How to improve]

## Suggestions 💡
Consider improving:

### 1. [File]: [Issue]
**Why:** [Reasoning]
**Benefit:** [What improves]

## Positive Aspects ✅
Good practices found:

- [Good practice 1]
- [Good practice 2]

## Test Coverage
- Current coverage: [X]%
- New lines covered: [X]%
- Missing coverage: [areas]

## Recommendations

### Immediate Actions
- [ ] Fix critical security issue in [file]
- [ ] Add error handling in [function]
- [ ] Add tests for [feature]

### Future Improvements
- [ ] Refactor [function] for better readability
- [ ] Consider extracting [logic] to separate module
- [ ] Add documentation for [component]

## Files Reviewed

### [filename1] - [X] issues
- [Issue 1]
- [Issue 2]

### [filename2] - [Y] issues
- [Issue 1]

## Overall Assessment

**Ready for merge?** [YES/NO/WITH CHANGES]

**Reasoning:** [Explanation]

**Risk Level:** [LOW/MEDIUM/HIGH]
```

### 4. Provide Actionable Fixes

For each critical or major issue, provide:

#### Specific Code Changes

```markdown
## Fix for [Issue]

**Current code:**
```language
[current problematic code]
```

**Suggested fix:**
```language
[improved code]
```

**Why this is better:**
[Explanation]
```

#### Implementation Steps

```markdown
1. Update [file] at line [X]
2. Add test case for [scenario]
3. Update documentation in [location]
4. Verify with [command]
```

### 5. Interactive Review Session

Ask the USER about fixes:

```markdown
## Issues Found: 3 critical, 5 warnings

Would you like me to:
1. Fix all critical issues automatically?
2. Review each issue one by one?
3. Focus on specific file/concern?
4. Generate fix suggestions only?
```

### 6. Apply Fixes (if requested)

If USER approves, systematically fix issues:

```markdown
## Fixing Issue 1/3: SQL Injection Risk

📝 Updating [file]
✅ Added parameterized query
✅ Added input validation
🧪 Added test case
✓ Issue resolved
```

## Review Categories

### Security Review

Focus on security vulnerabilities:
- Input validation
- Authentication/authorization
- Data encryption
- API security
- Dependency vulnerabilities

### Performance Review

Focus on performance:
- Algorithm efficiency
- Database queries
- Memory usage
- Network calls
- Caching opportunities

### Maintainability Review

Focus on code quality:
- Code readability
- Documentation
- Test coverage
- Error handling
- Code structure

### Style Review

Focus on consistency:
- Code formatting
- Naming conventions
- File organization
- Comment style
- Import order

## Customization

### Focus on Specific Aspects

```bash
/code-review-assistant --focus=security
/code-review-assistant --focus=performance
/code-review-assistant --focus=tests
```

### Review Specific Files

```bash
/code-review-assistant --files=src/api/*.ts
```

### Set Severity Threshold

```bash
/code-review-assistant --min-severity=warning
```

## Best Practices

✅ **Do:**
- Review changes before creating PR
- Address critical issues immediately
- Consider all suggestions
- Document complex changes
- Add tests for fixes

❌ **Don't:**
- Ignore security warnings
- Skip testing after fixes
- Rush through the review
- Dismiss all suggestions
- Forget to commit fixes

## Integration with Other Workflows

### Pre-PR Workflow

```bash
/run-tests-and-fix
/code-review-assistant
/commit-and-pr
```

### Post-Fix Workflow

```bash
/address-pr-comments
/code-review-assistant
/run-tests-and-fix
```

## Example Output

```markdown
# Code Review: Feature/user-auth

## Summary
✅ 8 files reviewed
🔴 1 critical issue
🟡 3 warnings
💡 5 suggestions

## Critical: Password Stored in Plain Text

**File:** src/auth/user.ts
**Line:** 45

**Problem:**
```typescript
user.password = req.body.password; // ❌ No hashing!
```

**Fix:**
```typescript
user.password = await bcrypt.hash(req.body.password, 10);
```

## Ready for Merge: NO
**Action Required:** Fix critical security issue

**After fixing:** Run tests and re-review
```

## Success Criteria

- [ ] All files reviewed
- [ ] Critical issues identified
- [ ] Actionable recommendations provided
- [ ] Fixes applied (if requested)
- [ ] Ready for human review

## Related Workflows

- `/run-tests-and-fix` - Ensure tests pass
- `/security-scan` - Deeper security analysis
- `/lint-and-format` - Code style cleanup
- `/commit-and-pr` - Create PR after review

---

**This workflow catches issues early and improves code quality before human review!**
