# pr-description skill

Generate a pull request description from the current git diff.

## Trigger

Use this skill when the user runs `/pr-description` or asks to "generate a PR description", "write a PR description", or "describe this PR".

## Instructions

### Step 1: Get the diff

Run the following to determine what changed:

```bash
git diff main 2>/dev/null
```

If that returns nothing (no commits on main yet, or branch is main), fall back to:

```bash
git diff --staged
```

If that also returns nothing, try:

```bash
git diff HEAD~1 2>/dev/null
```

Also collect commit messages for context:

```bash
git log main..HEAD --oneline 2>/dev/null || git log --oneline -10 2>/dev/null
```

### Step 2: Parse the diff

Analyze the diff output to identify:

1. **Files changed** — group by area (e.g., `src/auth/`, `tests/`, config files)
2. **Commit type** — infer from commit messages first; fall back to diff patterns described in `references/conventional-commits.md`
3. **Breaking changes** — look for `BREAKING CHANGE:` in commits, removed exports, renamed public APIs, changed function signatures
4. **What actually changed** — summarize the intent of each change, not just the mechanics

### Step 3: Generate the PR description

Output exactly the following markdown structure. Check the correct box for the detected commit type (replace `[ ]` with `[x]`). If multiple types apply, check all that match.

```markdown
## Summary
{1-2 sentences describing what this PR does and why}

## Type of change
- [ ] feat: new feature
- [ ] fix: bug fix
- [ ] refactor: code change that doesn't add features or fix bugs
- [ ] chore: dependency updates, config, tooling
- [ ] docs: documentation only
- [ ] perf: performance improvement
- [ ] test: adding or updating tests
- [ ] ci: CI/CD changes

## What changed
{Bullet points grouped by file or feature area. Be specific about what changed, not just that a file changed.}

## How to test
{Numbered steps a reviewer would take to verify the changes work correctly}

## Breaking changes
{List breaking changes, or write "None"}
```

### Guidelines

- Write the Summary from the perspective of a reviewer seeing this PR for the first time
- In "What changed", group related files together (e.g., group test files with the code they test)
- In "How to test", be concrete — include commands to run, UI interactions to try, or API calls to make
- Do not include the diff itself in the output
- If the diff is empty, tell the user: "No changes detected. Make sure you have uncommitted changes or commits ahead of main."
- Reference `references/conventional-commits.md` for type detection rules
