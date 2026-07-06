# Safe Git Workflow: Fork → Upstream → Dev → Main

## Objective

This practice demonstrates a safe Git workflow for making a small CI/CD configuration change in a forked repository and promoting it through the required branch flow.

The main objectives are:

- Keep the feature branch clean.
- Synchronize the fork with the upstream repository.
- Make only the required change.
- Validate changes before committing.
- Promote changes through Pull Requests.
- Avoid destructive Git operations.

---

## Repository Structure

We use two Git remotes:

```text
origin   → Forked repository
upstream → Original repository
```

Verify the configured remotes:

```bash
git remote -v
```

## Git Rules Followed

The following commands are intentionally avoided:

- `git pull`
- `git rebase`
- `git reset`
- `git push --force`

Instead, we use:

- `git fetch`
- `git merge`
- Fresh feature branches
- Normal `git push`
- Pull Requests for controlled promotion

---

## Step 1: Sync Main with Upstream

Switch to the local main branch:

```bash
git checkout main
```

Fetch the latest upstream changes:

```bash
git fetch upstream
```

Merge the latest upstream main into local main:

```bash
git merge upstream/main
```

Verify the state:

```bash
git status
git log --oneline --decorate -5
```

Push the synchronized main branch to the fork:

```bash
git push origin main
```

Expected state:

```
local main = origin/main = upstream/main
```

---

## Step 2: Create a Fresh Feature Branch

Create a new feature branch from the synchronized main branch:

```bash
git checkout -b feature/example-ci-change
```

Verify:

```bash
git branch --show-current
git status
```

Expected:

```
feature/example-ci-change
```

---

## Step 3: Push the Empty Branch to Origin

Push the unchanged feature branch to the fork:

```bash
git push -u origin feature/example-ci-change
```

This creates the feature branch in the fork before any code changes are made.

---

## Step 4: Push the Same Empty Branch to Upstream

Push the same branch to the upstream repository:

```bash
git push -u upstream feature/example-ci-change
```

The same branch now exists in:

- Local
- Origin
- Upstream

### Important

Using `-u` for the upstream push may change the branch's tracking configuration.

Therefore, use explicit remote names for later pushes:

```bash
git push origin feature/example-ci-change
```

or:

```bash
git push upstream feature/example-ci-change
```

---

## Step 5: Modify Only the Required File

Example workflow file:

```
.github/workflows/example-workflow.yml
```

Make only the required configuration change.

Best practices:

- Do not modify unrelated files.
- Do not hardcode credentials.
- Use repository secrets for sensitive values.
- Keep environment-specific configuration separate.
- Preserve existing deployment logic unless a change is required.

---

## Step 6: Validate Before Committing

Check for whitespace errors:

```bash
git diff --check
```

Expected result:

```
No output
```

Review only the intended workflow file:

```bash
git diff -- .github/workflows/example-workflow.yml
```

Check the overall repository status:

```bash
git status
```

Before committing, verify:

- Only the expected file changed.
- Only the required logic changed.
- YAML indentation is correct.
- No secrets are visible.
- No unrelated changes are included.

---

## Step 7: Commit the Change

Stage only the intended file:

```bash
git add .github/workflows/example-workflow.yml
```

Create a focused commit:

```bash
git commit -m "Update CI workflow configuration"
```

Verify the commit:

```bash
git show --stat --oneline HEAD
```

---

## Step 8: Push the Change to the Fork

Push explicitly to origin:

```bash
git push origin feature/example-ci-change
```

Using the explicit remote name prevents accidental pushes to the wrong repository.

---

## Pull Request Flow

The complete promotion flow is:

```
upstream/main
      ↓
Sync local main
      ↓
Sync fork main
      ↓
Create feature branch
      ↓
Push empty branch to origin
      ↓
Push same empty branch to upstream
      ↓
Make required change
      ↓
Validate the diff
      ↓
Commit
      ↓
Push change to origin feature branch
      ↓
PR: Origin Feature Branch → Upstream Feature Branch
      ↓
Merge
      ↓
PR: Upstream Feature Branch → Upstream Dev
      ↓
Test and validate in Dev
      ↓
Promote according to the repository release process
```

---

## Important Lesson: Avoid Mixing Divergent Branch Histories

If a feature branch is created from `main`, avoid directly merging a highly divergent `dev` branch into it unless that merge is explicitly required by the repository strategy.

Example of a risky operation:

```bash
git merge upstream/dev
```

This may bring a large number of unrelated commits into a small feature branch.

Possible problems:

- Noisy branch history
- Unrelated code in the Pull Request
- Difficult code review
- Increased conflict risk
- Loss of feature isolation

Preferred approach:

```
Create a clean feature branch from the required base
        ↓
Make only the required change
        ↓
Use the approved PR flow for promotion
```

---

## Useful Verification Commands

```bash
git remote -v

git branch --show-current

git status

git log --oneline --decorate -5

git diff --check

git diff -- .github/workflows/example-workflow.yml

git show --stat --oneline HEAD
```

---

## Security Best Practices

Never expose:

- Personal Access Tokens
- Webhook URLs
- Cloud credentials
- Private keys
- API keys

Do not store authentication tokens directly in Git remote URLs.

If a credential is exposed in:

- Terminal output
- Logs
- Screenshots
- Chat messages

Rotate or revoke it immediately.

---

## Final Checklist

- [ ] Upstream changes fetched successfully
- [ ] Local main synchronized with upstream/main
- [ ] Fork main synchronized
- [ ] Feature branch created from the correct base
- [ ] Empty branch pushed to origin
- [ ] Same empty branch pushed to upstream
- [ ] Only the required file changed
- [ ] No credentials hardcoded
- [ ] `git diff --check` passed
- [ ] Diff reviewed before commit
- [ ] Only the intended file staged
- [ ] Focused commit created
- [ ] Explicit remote used for push
- [ ] No force push used
- [ ] No rebase used
- [ ] No reset used
- [ ] No `git pull` used

---

## Key Takeaways

- Fetch remote changes before integrating them.
- Create feature branches from the correct clean base.
- Keep changes small and focused.
- Review the diff before every commit.
- Never hardcode credentials.
- Use explicit remote names when a branch exists on multiple remotes.
- Avoid mixing unrelated branch histories.
- Promote changes through controlled Pull Requests.
- Keep shared Git history intact.
- Validate every change before pushing.
