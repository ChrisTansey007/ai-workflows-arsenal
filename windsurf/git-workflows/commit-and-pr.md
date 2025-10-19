---
id: wkf.commit-and-pr
type: workflow
title: Commit and Create PR
tags: [git, pr, commit, workflow, automation]
summary: Creates standardized git commits with conventional commit messages and generates comprehensive pull requests with proper descriptions.
version: 1
---

# Commit and Create PR

**Standardized git commits and pull request creation**

## Description

This workflow helps you create well-formatted commits and pull requests with conventional commit messages, proper descriptions, and all necessary metadata.

## Use Case

Use this when:
- Ready to commit changes
- Want to create a pull request
- Need consistent commit messages
- Want proper PR documentation

## Prerequisites

- Git repository initialized
- Changes ready to commit
- GitHub CLI (`gh`) installed (for PR creation)

## Usage

```bash
/commit-and-pr
```

## Workflow Steps

### 1. Review Changes

Check what's changed:

```bash
git status
git diff
```

Identify:
- New files
- Modified files
- Deleted files
- Untracked files

### 2. Stage Changes

Stage files to commit:

**Stage all changes:**
```bash
git add .
```

**Stage specific files:**
```bash
git add [file1] [file2]
```

**Stage by pattern:**
```bash
git add src/**/*.ts
```

**Interactive staging:**
```bash
git add -p
```

### 3. Generate Commit Message

Create a conventional commit message:

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `style` - Code style (formatting, etc.)
- `refactor` - Code refactoring
- `perf` - Performance improvements
- `test` - Adding or updating tests
- `build` - Build system changes
- `ci` - CI configuration changes
- `chore` - Other changes

**Example:**
```
feat(auth): add password reset functionality

- Add password reset email endpoint
- Create password reset token generation
- Implement password update flow
- Add rate limiting to prevent abuse

Closes #123
```

### 4. Analyze Changes for Message

Automatically analyze changes to suggest commit message:

#### Detect Change Type

```markdown
**Changes detected:**
- 3 new files in src/auth/
- Modified authentication service
- Added 5 new tests

**Suggested type:** feat
**Suggested scope:** auth
**Suggested subject:** implement password reset flow
```

#### Generate Comprehensive Body

List all significant changes:
```markdown
**Body suggestions:**
- Add password reset email endpoint
- Create reset token with 1-hour expiration
- Implement secure password update flow
- Add rate limiting (5 attempts per hour)
- Add email template for reset instructions
- Update API documentation
```

#### Add Footer Information

```markdown
**Footer suggestions:**
- Closes #123
- Fixes #456
- Breaking change: requires email service configuration
- Reviewed-by: @reviewer
```

### 5. Create Commit

Execute the commit:

```bash
git commit -m "feat(auth): implement password reset flow

- Add password reset email endpoint
- Create reset token with 1-hour expiration
- Implement secure password update flow
- Add rate limiting (5 attempts per hour)

Closes #123"
```

### 6. Prepare PR Description

Generate comprehensive PR description:

```markdown
## Description

Brief summary of changes and why they were made.

## Type of Change

- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)
- [x] ✨ New feature (non-breaking change which adds functionality)
- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] 📝 Documentation update
- [ ] 🎨 Style update (formatting, renaming)
- [ ] ♻️ Code refactoring
- [ ] ⚡ Performance improvements
- [ ] ✅ Test updates
- [ ] 🔧 Build/CI updates
- [ ] 🔒 Security improvements

## Changes

- Implemented password reset email endpoint (`POST /auth/reset-password`)
- Created secure token generation with 1-hour expiration
- Added password update flow with token validation
- Implemented rate limiting (5 attempts per hour per IP)
- Added email template for reset instructions

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed
- [ ] All tests passing

### Test Coverage
- New endpoint: 100%
- Token generation: 100%
- Rate limiting: 95%

### Manual Testing Steps
1. Request password reset for existing user
2. Verify email received with reset link
3. Click link and enter new password
4. Confirm can login with new password
5. Verify old password no longer works

## Screenshots/Videos

[If applicable, add screenshots or videos demonstrating the changes]

## Related Issues

Closes #123

## Breaking Changes

- [ ] This PR introduces breaking changes
- [ ] Migration guide included

## Checklist

- [x] My code follows the project's style guidelines
- [x] I have performed a self-review of my code
- [x] I have commented my code, particularly in hard-to-understand areas
- [x] I have made corresponding changes to the documentation
- [x] My changes generate no new warnings
- [x] I have added tests that prove my fix is effective or that my feature works
- [x] New and existing unit tests pass locally with my changes
- [x] Any dependent changes have been merged and published

## Additional Context

Email service configuration required:
```env
EMAIL_SERVICE_API_KEY=your_key_here
EMAIL_FROM_ADDRESS=noreply@example.com
```

Rate limiting uses Redis for tracking attempts.

## Reviewer Notes

Please pay special attention to:
- Token generation security
- Rate limiting implementation
- Email template rendering
```

### 7. Create Pull Request

Use GitHub CLI to create PR:

```bash
gh pr create \
  --title "feat(auth): implement password reset flow" \
  --body "$(cat PR_DESCRIPTION.md)" \
  --base main \
  --head feature/password-reset \
  --assignee @me \
  --label enhancement \
  --label security
```

**Options:**
- `--title` - PR title (same as commit subject)
- `--body` - PR description
- `--base` - Target branch (usually main/master)
- `--head` - Source branch
- `--assignee` - Assign to someone
- `--reviewer` - Request reviewers
- `--label` - Add labels
- `--draft` - Create as draft PR

### 8. Provide Summary

After creating PR:

```markdown
## Commit & PR Created Successfully! ✅

**Commit:**
- Hash: abc123def
- Message: "feat(auth): implement password reset flow"
- Files changed: 8
- Insertions: 245 (+)
- Deletions: 12 (-)

**Pull Request:**
- Number: #456
- URL: https://github.com/user/repo/pull/456
- Status: Open
- Checks: Running...

**Next Steps:**
- [ ] Monitor CI/CD pipeline
- [ ] Address any failing checks
- [ ] Respond to review feedback
- [ ] Merge when approved

**PR URL:** https://github.com/user/repo/pull/456
```

## Best Practices

### Commit Message Guidelines

✅ **Do:**
- Use present tense ("add feature" not "added feature")
- Limit subject line to 50 characters
- Capitalize subject line
- No period at end of subject
- Wrap body at 72 characters
- Separate subject from body with blank line
- Explain what and why, not how

❌ **Don't:**
- Vague messages ("fix bug", "update code")
- Multiple unrelated changes in one commit
- Giant commits with many files
- Missing context or reasoning

### PR Description Guidelines

✅ **Do:**
- Explain the "why" behind changes
- Link to related issues
- Include testing information
- Add screenshots for UI changes
- Document breaking changes
- Provide migration steps if needed

❌ **Don't:**
- Empty or minimal descriptions
- Assume reviewers know context
- Skip testing details
- Forget to link issues

## Commit Message Templates

### Feature

```
feat(scope): add feature X

- Implement core functionality
- Add error handling
- Include documentation

Closes #123
```

### Bug Fix

```
fix(scope): resolve issue with Y

- Identify root cause
- Apply fix
- Add regression test

Fixes #456
```

### Documentation

```
docs(scope): update documentation

- Add API examples
- Fix typos
- Update installation guide
```

### Refactoring

```
refactor(scope): improve code structure

- Extract common logic
- Simplify complex functions
- Remove duplication
```

## Customization

### Custom Commit Template

Create `.gitmessage` file:
```
<type>(<scope>): <subject>

# Why this change?

# What changed?

# Breaking changes?

# Related issues:
```

Use it:
```bash
git config commit.template .gitmessage
```

### PR Template

Create `.github/pull_request_template.md` in your repo.

### Auto-assign Reviewers

```bash
gh pr create --reviewer @teammate1,@teammate2
```

## Integration

### Full Workflow

```bash
# 1. Review code first
/code-review-assistant

# 2. Fix any issues
/run-tests-and-fix

# 3. Commit and create PR
/commit-and-pr
```

### After PR Created

```bash
# Monitor CI
gh pr checks

# View PR
gh pr view

# Update PR
gh pr edit [number]
```

## Troubleshooting

### Issue: Can't push commits

**Solution:**
```bash
git pull --rebase origin main
git push
```

### Issue: Wrong branch

**Solution:**
```bash
git checkout -b correct-branch
git cherry-pick [commit-hash]
```

### Issue: Need to amend commit

**Solution:**
```bash
git add [forgotten-files]
git commit --amend --no-edit
git push --force-with-lease
```

## Success Criteria

- [ ] Changes staged correctly
- [ ] Commit message follows conventions
- [ ] PR description is comprehensive
- [ ] Related issues linked
- [ ] Labels and assignees added
- [ ] PR created successfully

## Related Workflows

- `/code-review-assistant` - Review before committing
- `/run-tests-and-fix` - Ensure tests pass
- `/address-pr-comments` - Handle feedback
- `/changelog-generator` - Update changelog

---

**This workflow ensures consistent, well-documented commits and PRs!**
