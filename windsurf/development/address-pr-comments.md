# Address PR Comments

**Automatically process and implement Pull Request feedback**

## Description

This workflow helps you efficiently address PR comments by fetching all feedback, analyzing the requested changes, and implementing them systematically.

## Use Case

Use this when:
- You have PR comments to address
- You want to ensure all feedback is implemented
- You need to track which comments have been addressed
- You want AI assistance in implementing changes

## Prerequisites

- GitHub CLI (`gh`) installed and authenticated
- PR must be accessible
- You have permission to make changes to the branch

## How It Works

This workflow will:
1. Check out the PR branch locally
2. Fetch all comments from the PR
3. Analyze each comment and the related code
4. Implement requested changes
5. Summarize completed work

## Usage

```bash
/address-pr-comments
```

Then provide the PR number when prompted.

## Workflow Steps

### 1. Checkout the PR Branch

First, check out the branch associated with the PR:

```bash
gh pr checkout [PR_NUMBER]
```

Replace `[PR_NUMBER]` with the actual PR number.

### 2. Fetch PR Comments

Retrieve all comments from the PR with detailed information:

```bash
gh api --paginate repos/[OWNER]/[REPO]/pulls/[PR_NUMBER]/comments | \
  jq '.[] | {user: .user.login, body, path, line, original_line, created_at, in_reply_to_id, pull_request_review_id, commit_id}'
```

**Replace:**
- `[OWNER]` - Repository owner
- `[REPO]` - Repository name
- `[PR_NUMBER]` - PR number

### 3. Process Each Comment

For EVERY comment, do the following (one at a time):

#### a. Display the Comment

Output: `"(index). From [user] to [file]:[lines] — [body]"`

Example: `"1. From reviewer123 to src/app.ts:45-52 — This function should handle null cases"`

#### b. Analyze the Code

- Read the relevant file
- Review the specific line range
- Understand the current implementation
- Identify what change is being requested

#### c. Decide on Action

**If the comment is unclear:**
- Do NOT make changes
- Ask the USER for clarification
- Move to next comment

**If you understand the request:**
- Explain what change you'll make
- Implement the change
- Test the change (if applicable)
- COMPLETE the change BEFORE moving to the next comment

#### d. Implement the Change

- Make the requested modification
- Follow code style of the file
- Add comments if helpful
- Ensure tests still pass (if applicable)

### 4. Final Summary

After processing ALL comments, provide a summary:

```markdown
## PR Comments Addressed

### Completed Changes
1. [File]: [What was changed]
2. [File]: [What was changed]
...

### Needs USER Attention
1. [Comment]: [Why it needs manual review]
2. [Comment]: [Why it needs manual review]
...

### Next Steps
- [ ] Review the changes
- [ ] Run tests locally
- [ ] Push changes to the PR branch
- [ ] Respond to reviewers
```

## Important Notes

- **Process comments ONE AT A TIME** - Don't skip ahead
- **Complete each change** before moving to the next
- **Ask for help** if a comment is ambiguous
- **Test your changes** when possible
- **Commit frequently** as you make progress

## Example Usage

### Scenario: PR with 3 comments

```markdown
Comment 1: "Add error handling for null values"
Action: Add try-catch block and null checks

Comment 2: "Extract this logic into a separate function"
Action: Create new function and refactor

Comment 3: "Update the test to cover edge case"
Action: Add test case for edge condition
```

### Expected Output:

```markdown
## Processing PR Comments

### Comment 1/3
From: reviewer123
File: src/utils.ts:45
Request: "Add error handling for null values"

✅ Implemented: Added try-catch block and null validation
```

## Customization

### Process Only Specific Comments

If you only want to address certain comments:

```bash
/address-pr-comments --filter="[username]"
```

### Skip Already Resolved

Track which comments you've addressed:

```bash
/address-pr-comments --skip-resolved
```

## Troubleshooting

### Issue: Can't fetch PR comments

**Solution:** Ensure you're authenticated with `gh auth login`

### Issue: Don't understand a comment

**Solution:** Ask the USER for clarification instead of guessing

### Issue: Change breaks tests

**Solution:** Revert the change and ask for guidance

## Related Workflows

- `/run-tests-and-fix` - Run tests after addressing comments
- `/code-review-assistant` - Pre-review before pushing
- `/commit-and-pr` - Commit and update the PR

## Tips

✅ **Do:**
- Read each comment carefully
- Test changes locally
- Commit incrementally
- Ask questions when unsure

❌ **Don't:**
- Rush through comments
- Make assumptions
- Skip testing
- Batch unrelated changes

## Success Criteria

- [ ] All comments reviewed
- [ ] Requested changes implemented
- [ ] Tests passing
- [ ] Ready to push to PR branch
- [ ] Summary provided to USER

---

**This workflow streamlines PR feedback implementation and ensures nothing is missed!**
