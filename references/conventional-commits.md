# Conventional Commits Reference

Conventional Commits is a specification for adding human and machine-readable meaning to commit messages. Format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Commit Types

| Type | Description | PR Label |
|------|-------------|----------|
| `feat` | A new feature for the user | New feature |
| `fix` | A bug fix for the user | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature | Refactoring |
| `chore` | Build process, dependency updates, tooling, config changes | Maintenance |
| `docs` | Documentation only changes | Documentation |
| `perf` | A code change that improves performance | Performance |
| `test` | Adding missing tests or correcting existing tests | Testing |
| `ci` | Changes to CI/CD configuration files and scripts | CI/CD |
| `style` | Changes that do not affect meaning (whitespace, formatting) | Style |
| `revert` | Reverts a previous commit | Revert |
| `build` | Changes that affect the build system or external dependencies | Build |

## Breaking Changes

A breaking change is indicated by:
1. A `!` after the type/scope: `feat!: remove deprecated API`
2. A `BREAKING CHANGE:` footer in the commit body

Breaking changes require a major version bump and must be documented.

## Detecting Type from Diff

When commit messages are not available or unclear, infer type from the diff:

- **feat**: New files, new exported functions/classes, new API endpoints, new UI components
- **fix**: Changes to conditionals, error handling, off-by-one corrections, null checks
- **refactor**: Renamed symbols, restructured logic with same behavior, extracted helpers
- **chore**: `package.json`, `*.lock`, `.github/`, config files, `.env.example`
- **docs**: `*.md`, `*.txt`, `*.rst`, JSDoc/docstring additions, comment-only changes
- **perf**: Caching, memoization, algorithm improvements, reduced iterations
- **test**: `*.test.*`, `*.spec.*`, `__tests__/`, test fixtures, test helpers
- **ci**: `.github/workflows/`, `Dockerfile`, `docker-compose.yml`, `.travis.yml`, `Jenkinsfile`

## Scope

The scope is an optional noun describing the section of the codebase:
- `feat(auth): add OAuth login`
- `fix(api): handle empty response`
- `chore(deps): bump lodash to 4.17.21`

## Examples

```
feat(auth): add Google OAuth login

Users can now sign in with their Google account via OAuth 2.0.

Closes #42
```

```
fix: prevent race condition in data fetch

Added debounce to prevent multiple simultaneous API calls
when the user types quickly in the search box.
```

```
feat!: rename `user_id` to `userId` in all API responses

BREAKING CHANGE: All API responses now use camelCase.
Clients must update their field references.
```
