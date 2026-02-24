# Contributing

Thanks for wanting to improve pr-description! Here's how to get started.

## What to contribute

**Good first issues:**
- Improving type detection heuristics (e.g., detecting `refactor` vs `feat` better)
- Adding examples to `references/conventional-commits.md`
- Expanding the "How to test" section with more realistic scenarios
- Bug reports — if the skill generates incorrect output, open an issue with the diff

**Feature ideas:**
- Custom output templates (e.g., GitHub-flavored, GitLab-flavored)
- Support for detecting `BREAKING CHANGE` footers
- Better monorepo support (grouping changes by package)
- Scope detection from commit messages (e.g., `feat(auth):` → include scope in summary)

## How to contribute

1. **Fork and clone**
   ```bash
   git clone https://github.com/bidemiajala/pr-description
   cd pr-description
   ```

2. **Make your changes**
   - Edit `SKILL.md` if changing how the skill works
   - Edit `references/conventional-commits.md` if updating type detection rules
   - Edit `README.md` if documentation needs updating

3. **Test your changes**
   - Read through your edits for clarity and correctness
   - If you changed type detection, verify it still handles the examples in `references/conventional-commits.md`

4. **Open a pull request**
   - Link any related issues
   - Describe what you changed and why
   - Use the pr-description skill itself to write your PR description 😉

## Code of conduct

Be respectful. We're all here to make PR writing easier.

## Questions?

Open an issue or discussion. No such thing as a dumb question.
