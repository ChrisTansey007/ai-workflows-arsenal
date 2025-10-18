# Run Tests and Fix

**Automatically run tests, identify failures, and fix them**

## Description

This workflow runs your test suite, analyzes any failures, and attempts to fix them automatically by understanding the error messages and modifying the relevant code.

## Use Case

Use this when:
- Tests are failing after changes
- You want to quickly fix test failures
- You need to understand why tests broke
- You want AI help debugging test issues

## Prerequisites

- Test suite configured in your project
- Test command available (npm test, pytest, etc.)
- Tests are in a runnable state

## Usage

```bash
/run-tests-and-fix
```

## Workflow Steps

### 1. Identify Test Command

Determine the correct test command for your project:

**Node.js/JavaScript:**
```bash
npm test
# or
yarn test
# or
pnpm test
```

**Python:**
```bash
pytest
# or
python -m pytest
# or
python -m unittest
```

**Ruby:**
```bash
rake test
# or
rspec
```

**Go:**
```bash
go test ./...
```

**Rust:**
```bash
cargo test
```

### 2. Run Tests

Execute the test suite and capture output:

```bash
[TEST_COMMAND] 2>&1 | tee test-output.log
```

Capture:
- Exit code (0 = pass, non-zero = fail)
- Full output including errors
- Failed test names
- Error messages and stack traces

### 3. Analyze Failures

For each failed test:

#### a. Parse Test Output

Extract:
- Test name and location
- Error type (assertion, exception, timeout, etc.)
- Error message
- Stack trace
- Expected vs actual values

#### b. Locate Source Code

Find the relevant files:
- Test file itself
- Code being tested
- Any imported dependencies

#### c. Understand the Failure

Analyze:
- What the test is checking
- Why it's failing now
- What changed recently
- What the fix should be

### 4. Fix Each Failure

For EACH failed test:

#### a. Display Failure Info

```markdown
## Fixing Test [N]/[TOTAL]: [TEST_NAME]

**Location:** [file]:[line]
**Error:** [error message]
**Type:** [assertion/exception/timeout]
```

#### b. Propose Fix

Explain:
- Root cause of the failure
- Proposed solution
- Files that need changes
- Potential side effects

#### c. Implement Fix

Choose the appropriate fix:

**If it's a code bug:**
- Fix the implementation
- Ensure logic matches test expectations

**If it's a test bug:**
- Update test expectations
- Fix test setup/teardown
- Correct test assertions

**If it's an environment issue:**
- Update test configuration
- Fix dependencies
- Adjust test conditions

#### d. Verify Fix

Run the specific test again:
```bash
[TEST_COMMAND] [SPECIFIC_TEST]
```

Confirm it passes before moving to next test.

### 5. Run Full Suite Again

After fixing all tests, run the complete suite:

```bash
[TEST_COMMAND]
```

Verify:
- All previously failing tests now pass
- No new failures introduced
- Test suite is green

### 6. Summary Report

Provide a detailed summary:

```markdown
## Test Fixing Summary

### Initial Status
- Total tests: [N]
- Failed: [N]
- Passed: [N]

### Fixes Applied
1. **[Test Name]**
   - Issue: [Description]
   - Fix: [What was done]
   - File: [Changed file]

2. **[Test Name]**
   - Issue: [Description]
   - Fix: [What was done]
   - File: [Changed file]

### Final Status
- Total tests: [N]
- Failed: [N]
- Passed: [N]
- Success rate: [percentage]

### Files Modified
- [file1] - [description]
- [file2] - [description]

### Recommendations
- [ ] Review all changes
- [ ] Commit with descriptive message
- [ ] Consider adding more test coverage
- [ ] Update documentation if behavior changed
```

## Important Guidelines

### When to Fix Code vs. Tests

**Fix the code when:**
- Test expectations are correct
- Code doesn't match specification
- Logic error exists
- Edge cases not handled

**Fix the test when:**
- Test has incorrect assertions
- Test setup is wrong
- Flaky test due to timing
- Test expectations outdated

### Safety Checks

Before making changes:
- [ ] Understand what the test validates
- [ ] Verify the test was passing before
- [ ] Check if recent changes broke it
- [ ] Consider if fix has side effects

### When to Ask for Help

**Don't fix automatically if:**
- Unsure about correct behavior
- Test failure seems intentional
- Major architectural changes needed
- Multiple conflicting solutions possible

## Example Scenarios

### Scenario 1: Assertion Failure

```
FAILED tests/test_math.py::test_addition
Expected: 5
Got: 6
```

**Fix:** Check addition function, fix off-by-one error

### Scenario 2: Exception

```
FAILED tests/test_api.py::test_fetch_user
TypeError: 'NoneType' object is not subscriptable
```

**Fix:** Add null check before accessing object properties

### Scenario 3: Timeout

```
FAILED tests/test_async.py::test_data_fetch
Timeout after 5000ms
```

**Fix:** Optimize query or increase timeout threshold

## Customization

### Run Specific Test File

```bash
/run-tests-and-fix --file=tests/test_api.py
```

### Run with Coverage

```bash
/run-tests-and-fix --coverage
```

### Verbose Mode

```bash
/run-tests-and-fix --verbose
```

## Best Practices

✅ **Do:**
- Run tests before starting fixes
- Fix one test at a time
- Verify each fix individually
- Maintain test quality
- Document complex fixes

❌ **Don't:**
- Skip test failures
- Make tests always pass artificially
- Remove failing tests
- Ignore underlying issues
- Batch fixes without verification

## Integration

### After Addressing PR Comments

```bash
/address-pr-comments
/run-tests-and-fix
```

### Before Creating PR

```bash
/run-tests-and-fix
/code-review-assistant
/commit-and-pr
```

## Troubleshooting

### Issue: Tests won't run

**Solution:** Check test command and dependencies installed

### Issue: Too many failures

**Solution:** Fix critical failures first, then re-run

### Issue: Flaky tests

**Solution:** Identify timing issues, add proper waits

### Issue: Environment-specific failures

**Solution:** Check environment variables and configuration

## Success Criteria

- [ ] All tests passing
- [ ] No new test failures introduced
- [ ] Fixes are logical and correct
- [ ] Code quality maintained
- [ ] Ready to commit changes

## Related Workflows

- `/address-pr-comments` - Fix PR feedback first
- `/code-review-assistant` - Review changes before commit
- `/security-scan` - Run security checks
- `/lint-and-format` - Clean up code style

---

**This workflow ensures your test suite stays green and catches issues early!**
