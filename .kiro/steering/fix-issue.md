# Fix Issue Workflow

When asked to fix a bug or issue, follow this process:

## 1. Understand the Issue
- Read the error message or bug description carefully
- Identify the affected component(s)
- Reproduce the issue locally if possible

## 2. Root Cause Analysis
- Check recent git changes: `git log --oneline -20`
- Review affected files
- Look for related tests that may reveal expected behavior

## 3. Plan the Fix
- Identify the minimal change needed
- Consider side effects on other components
- Update or add tests to cover the fix

## 4. Implement
- Make the targeted fix
- Ensure code follows `code-style.md`
- Handle errors per `error-handling.md`

## 5. Verify
```bash
npm test -- --testPathPattern=[affected]
npm test
npm run lint
```

## 6. Commit
Follow `git-workflow.md`:
```
fix: [short description of the fix]

Closes #[issue-number]
```
