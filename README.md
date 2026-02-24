# pr-description

A Claude skill that generates structured pull request descriptions from your current git diff.

## What it does

Run `/pr-description` in any git repository and Claude will:

1. Read `git diff main` (or `git diff --staged` if you're on main or have no commits yet)
2. Analyze the changed files and commit messages
3. Detect the [conventional commit](https://www.conventionalcommits.org/) type (`feat`, `fix`, `chore`, etc.)
4. Output a ready-to-paste PR description in markdown

### Output format

```markdown
## Summary
A short 1-2 sentence description of what this PR does.

## Type of change
- [x] feat: new feature
- [ ] fix: bug fix
...

## What changed
- `src/auth/`: Added Google OAuth login flow
- `tests/`: Added integration tests for OAuth callback

## How to test
1. Run `npm test`
2. Navigate to /login and click "Sign in with Google"
...

## Breaking changes
None
```

## Installation

Copy this folder into `~/.claude/skills/`:

```bash
cp -r pr-description ~/.claude/skills/
```

Or clone it directly:

```bash
git clone <repo-url> ~/.claude/skills/pr-description
```

Restart Claude Code (or reload skills) after installing.

## Usage

In any git repository, run:

```
/pr-description
```

Claude will detect the diff automatically. No arguments needed.

## How type detection works

Claude reads commit messages first and matches against conventional commit prefixes (`feat:`, `fix:`, etc.). If commit messages are ambiguous, it falls back to analyzing the diff itself — for example, changes to `*.test.*` files indicate `test`, changes to `.github/workflows/` indicate `ci`.

See [`references/conventional-commits.md`](references/conventional-commits.md) for the full detection rules.

## File structure

```
pr-description/
├── SKILL.md                          # Skill instructions (loaded by Claude)
├── README.md                         # This file
└── references/
    └── conventional-commits.md       # Conventional commits spec for type detection
```
