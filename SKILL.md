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

If there are many commits on the branch, use `git log main..HEAD` (without `--oneline`) to read the full message bodies — they often contain context that improves the summary.

### Step 2: Detect the language and ecosystem

Before parsing the diff, identify what kind of project this is. This affects type detection, test commands, and "How to test" steps.

| Indicator files | Ecosystem |
|---|---|
| `package.json`, `*.ts`, `*.tsx`, `*.js` | JavaScript / TypeScript |
| `pyproject.toml`, `setup.py`, `requirements.txt`, `*.py` | Python |
| `go.mod`, `*.go` | Go |
| `Cargo.toml`, `*.rs` | Rust |
| `build.sbt`, `*.scala` | Scala / JVM |
| `pom.xml`, `build.gradle`, `*.java` | Java / JVM |
| `build.gradle.kts`, `*.kt`, `*.kts` | Kotlin |
| `*.rb`, `Gemfile` | Ruby |
| `*.swift`, `Package.swift` | Swift |
| `pubspec.yaml`, `*.dart` | Dart / Flutter |
| `composer.json`, `*.php` | PHP |
| `*.csproj`, `*.sln`, `*.cs` | C# / .NET |
| `CMakeLists.txt`, `Makefile`, `*.cpp`, `*.c`, `*.h` | C / C++ |
| `mix.exs`, `*.ex`, `*.exs` | Elixir |
| `*.cabal`, `stack.yaml`, `*.hs` | Haskell |
| `project.clj`, `deps.edn`, `*.clj`, `*.cljs` | Clojure |
| `*.cbl`, `*.cob`, `*.cpy` | COBOL |
| `*.tf`, `*.hcl` | Terraform / Infrastructure |
| `*.yaml`, `*.yml` in `.github/workflows/` | CI/CD (GitHub Actions) |

Use the detected ecosystem to choose the right test commands in Step 3.

### Step 3: Parse the diff

Analyze the diff output to identify:

1. **Files changed** — group by area (e.g., `src/auth/`, `tests/`, config files)
2. **Commit type** — infer from commit messages first; fall back to file-path patterns in `references/conventional-commits.md`
3. **Scope** — if present in commit messages (e.g., `feat(auth):`), include in summary context
4. **Breaking changes** — look for `BREAKING CHANGE:` in commits, removed exports, renamed public APIs, changed function signatures, removed CLI flags, changed config keys
5. **What actually changed** — summarize the intent of each change, not just the mechanics

#### Type detection by file path (language-aware)

**JavaScript / TypeScript**
- `package.json`, `*.lock`, `.nvmrc` → `chore`
- `*.test.ts`, `*.spec.ts`, `__tests__/` → `test`
- `.github/workflows/` → `ci`
- `*.md` → `docs`
- New exported function/class/hook → `feat`
- Bugfix in existing function body → `fix`

**Python**
- `pyproject.toml`, `requirements*.txt`, `setup.cfg` → `chore`
- `test_*.py`, `*_test.py`, `tests/` → `test`
- `*.md`, docstring-only changes → `docs`
- New function/class definition → `feat`
- Fix in conditional, exception handling → `fix`

**Go**
- `go.mod`, `go.sum` → `chore`
- `*_test.go` → `test`
- `.github/workflows/` → `ci`
- New exported function (`func [A-Z]`) → `feat`
- Fix to error handling, nil checks → `fix`

**Rust**
- `Cargo.toml`, `Cargo.lock` → `chore`
- `#[cfg(test)]` blocks, `tests/` → `test`
- New `pub fn`, `pub struct`, `pub trait` → `feat`
- Fix to `unwrap`, `expect`, error propagation → `fix`

**Scala / JVM**
- `build.sbt`, `pom.xml`, `build.gradle`, `*.conf` → `chore`
- `*Test.scala`, `*Spec.scala`, `*Suite.scala` → `test`
- New `class`, `object`, `trait`, `def` → `feat`
- Fix to pattern match, `Option` handling → `fix`

**Kotlin**
- `build.gradle.kts`, `*.kts` → `chore`
- `*Test.kt`, `*Spec.kt` → `test`
- New `fun`, `class`, `interface`, `object` → `feat`
- Fix to null safety (`?.`, `!!`), exception handling → `fix`

**Dart / Flutter**
- `pubspec.yaml`, `pubspec.lock` → `chore`
- `*_test.dart`, `test/` → `test`
- New `Widget`, `StatefulWidget`, `StatelessWidget` → `feat`
- Fix to `null` handling, async/await → `fix`

**PHP**
- `composer.json`, `composer.lock` → `chore`
- `*Test.php`, `tests/` → `test`
- New class, interface, trait → `feat`
- Fix to conditionals, exception handling → `fix`

**C# / .NET**
- `*.csproj`, `*.sln`, `NuGet.Config` → `chore`
- `*Tests.cs`, `*Test.cs`, `Tests/` → `test`
- New `public class`, `interface`, `record` → `feat`
- Fix to null checks, exception handling → `fix`
- Run tests: `dotnet test` | Run app: `dotnet run`

**C / C++**
- `CMakeLists.txt`, `Makefile`, `conanfile.txt` → `chore`
- `*_test.cpp`, `*_test.c`, `tests/` → `test`
- New function declaration in header → `feat`
- Fix to pointer handling, bounds checks, memory management → `fix`
- Run tests: `make test` or `ctest` | Build: `make` or `cmake --build .`

**Elixir**
- `mix.exs`, `mix.lock` → `chore`
- `*_test.exs`, `test/` → `test`
- New `def`, `defmodule`, `defmacro` → `feat`
- Fix to pattern match, error tuple handling → `fix`
- Run tests: `mix test` | Run app: `mix run` or `iex -S mix`

**Haskell**
- `*.cabal`, `stack.yaml`, `package.yaml` → `chore`
- `*Spec.hs`, `test/` → `test`
- New top-level function, typeclass instance → `feat`
- Fix to pattern match, `Maybe`/`Either` handling → `fix`
- Run tests: `stack test` or `cabal test` | Run app: `stack run`

**Clojure**
- `project.clj`, `deps.edn`, `shadow-cljs.edn` → `chore`
- `*_test.clj`, `test/` → `test`
- New `defn`, `defprotocol`, `defrecord` → `feat`
- Fix to nil handling, exception catching → `fix`
- Run tests: `lein test` or `clj -M:test` | Run app: `lein run`

**COBOL**
- `*.cbl`, `*.cob`, `*.cpy` are all source files — no separate test file convention; note test procedures inline
- Changes to `WORKING-STORAGE SECTION` → data structure change (`feat` or `refactor`)
- Changes to `PROCEDURE DIVISION` → logic change (`feat` or `fix`)
- Changes to `COPY` statements or copybooks (`*.cpy`) → `chore` or `refactor`
- Breaking changes: renamed `COPY` members, removed `01` level fields used by callers, changed `LINKAGE SECTION` interface
- Run/compile: `cobc -x program.cbl && ./program` (GnuCOBOL) or per-mainframe JCL

**Infrastructure / Terraform**
- `*.tf`, `*.tfvars` → `chore` or `feat` depending on whether it's new infra
- `*.yaml` in `.github/workflows/` → `ci`
- `Dockerfile`, `docker-compose.yml` → `chore` or `ci`

### Step 4: Generate the PR description

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

#### "How to test" commands by ecosystem

Use the right command for the project type:

| Ecosystem | Run tests | Start / build |
|---|---|---|
| JavaScript / TypeScript | `npm test` or `pnpm test` or `bun test` | `npm run dev` |
| Python | `pytest` or `python -m pytest` | `uvicorn main:app` or `python manage.py runserver` |
| Go | `go test ./...` | `go run .` |
| Rust | `cargo test` | `cargo run` |
| Scala (sbt) | `sbt test` | `sbt run` |
| Java (Maven) | `mvn test` | `mvn spring-boot:run` |
| Java (Gradle) | `./gradlew test` | `./gradlew bootRun` |
| Kotlin (Gradle) | `./gradlew test` | `./gradlew run` |
| Ruby | `bundle exec rspec` | `rails server` |
| Swift | `swift test` | `swift run` |
| Dart / Flutter | `flutter test` | `flutter run` |
| PHP | `./vendor/bin/phpunit` | `php artisan serve` (Laravel) or `php -S localhost:8000` |
| C# / .NET | `dotnet test` | `dotnet run` |
| C / C++ | `make test` or `ctest` | `make` or `cmake --build .` |
| Elixir | `mix test` | `iex -S mix` |
| Haskell | `stack test` or `cabal test` | `stack run` |
| Clojure | `lein test` or `clj -M:test` | `lein run` |
| COBOL (GnuCOBOL) | Manual procedure walkthrough | `cobc -x program.cbl && ./program` |

If multiple package managers are possible (e.g., `npm` vs `pnpm`), check for a lockfile: `pnpm-lock.yaml` → pnpm, `bun.lockb` → bun, `yarn.lock` → yarn, `package-lock.json` → npm.

### Common scenarios

**Scenario: dependency update only**
- All changes are in `package.json` / `go.mod` / `Cargo.toml` / `requirements.txt`
- Type: `chore`
- "How to test": install deps and run the test suite

**Scenario: new feature with tests**
- New source file + new test file
- Type: `feat` + `test`
- Group source and test files together in "What changed"

**Scenario: bug fix**
- Change is inside an existing function — conditional logic, null check, error handling
- Type: `fix`
- "How to test": describe the specific scenario that previously failed

**Scenario: config or infrastructure change**
- Changes to `.conf`, `.env.example`, `*.tf`, `docker-compose.yml`
- Type: `chore` (or `ci` if CI/CD related)
- "How to test": describe how to verify the config takes effect

**Scenario: large PR with many commits**
- Run `git log main..HEAD` (full messages) to understand intent across commits
- Summarize the overall goal in "Summary", not each individual commit
- Group "What changed" by feature area, not by commit

**Scenario: breaking change**
- Removed export, renamed public API, changed function signature, removed config key
- Check `BREAKING CHANGE:` in commit footers
- List specifics in "Breaking changes": what was removed/renamed and what replaces it

### Edge cases

- **On `main` with no upstream branch** — use `git diff --staged` to pick up staged changes; if nothing staged, use `git diff HEAD~1`
- **Merge commits in the diff** — ignore them; focus on the actual file changes
- **Binary files changed** (images, fonts) — note them briefly in "What changed" but don't describe the binary contents
- **Generated files** (e.g., `*.pb.go`, `package-lock.json`, `yarn.lock`, migration files) — note that they are auto-generated and skip detailed analysis of their contents
- **Whitespace-only changes** — classify as `style`, not `refactor`
- **Empty diff** — tell the user: "No changes detected. Make sure you have uncommitted changes or commits ahead of main."

### Output checklist (self-review before responding)

Before outputting the PR description, verify:

- [ ] Summary is 1-2 sentences and written from a reviewer's perspective
- [ ] At least one type checkbox is checked
- [ ] Every bullet in "What changed" corresponds to an actual file or group of files in the diff
- [ ] "How to test" steps use the correct commands for this project's ecosystem
- [ ] "Breaking changes" is either a specific list or "None" — never left blank

### Guidelines

- Write the Summary from the perspective of a reviewer seeing this PR for the first time
- In "What changed", group related files together (e.g., group test files with the code they test)
- In "How to test", be concrete — include commands to run, UI interactions to try, or API calls to make
- Do not include the diff itself in the output
- Do not mention the language or ecosystem explicitly unless it adds clarity
- Reference `references/conventional-commits.md` for full type detection rules
