# PR Description

> A Claude Code skill that writes your pull request descriptions for you — from diff to markdown in one command.

---

## The problem

Writing a good PR description takes time. You have to remember what you changed, why you changed it, and how someone else should test it. Most people either skip it or write something vague. Reviewers suffer.

This skill reads your actual git diff and does the writing for you.

---

## What it does

Run `/pr-description` in any git repository. Claude will:

1. Read `git diff main` (falling back to `git diff --staged` or `git diff HEAD~1` if needed)
2. Analyze the changed files and commit messages
3. Detect the [conventional commit](https://www.conventionalcommits.org/) type — `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf`, `ci`
4. Output a ready-to-paste PR description in markdown

### Sample output

```markdown
## Summary
Adds rate limiting to the public API to prevent abuse and reduce infrastructure costs.

## Type of change
- [ ] feat: new feature
- [ ] fix: bug fix
- [ ] refactor: code change that doesn't add features or fix bugs
- [x] chore: dependency updates, config, tooling
- [ ] docs: documentation only
- [ ] perf: performance improvement
- [ ] test: adding or updating tests
- [ ] ci: CI/CD changes

## What changed
- `src/middleware/`: Added `rateLimiter.ts` using `express-rate-limit`
- `src/routes/api.ts`: Applied rate limiter middleware to all public routes
- `tests/middleware/`: Added unit tests for limit thresholds and 429 responses

## How to test
1. Run `npm test` — all middleware tests should pass
2. Start the dev server: `npm run dev`
3. Send >100 requests/minute to any `/api/*` route and confirm a `429` response

## Breaking changes
None
```

---

## Installation

### Claude Code (recommended)

```bash
git clone https://github.com/bidemiajala/pr-description ~/.claude/skills/pr-description
```

Then reload skills in Claude Code.

### Manual

```bash
cp -r pr-description ~/.claude/skills/
```

---

## Usage

```
/pr-description
```

No arguments. No configuration. Claude figures out the diff automatically.

Works on any git repo — Node, Python, Go, Rust, monorepos, whatever you're building.

---

## How type detection works

Commit messages are read first. If they follow [conventional commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, etc.), the type is taken directly from the prefix.

If messages are absent or freeform, Claude falls back to file-path heuristics:

| Files changed | Detected type |
|---|---|
| `*.test.*`, `*.spec.*`, `__tests__/` | `test` |
| `.github/workflows/`, `Dockerfile` | `ci` |
| `*.md`, comment-only diffs | `docs` |
| `package.json`, `*.lock`, config files | `chore` |
| New exported functions, new routes | `feat` |
| Conditionals, null checks, error handling | `fix` |

Full rules: [`references/conventional-commits.md`](references/conventional-commits.md)

---

## Philosophy

PR descriptions are communication, not ceremony. A good one tells the reviewer what changed, why it matters, and how to verify it — without them having to read every line of diff.

This skill exists because that communication shouldn't require extra effort after the code is already written.

---

## File structure

```
pr-description/
├── SKILL.md                        # Skill instructions (read by Claude)
├── README.md                       # This file
└── references/
    └── conventional-commits.md     # Type detection reference
```

---

## Learn more

[Read the blog post](https://www.bidemi.xyz/blog/i-built-a-claude-skill-that-writes-your-pr-descriptions) about how and why this skill was built.

---

## Credits

Built by **Bidemi** with [Claude Code](https://claude.ai).

---

## License

[MIT](LICENSE)
