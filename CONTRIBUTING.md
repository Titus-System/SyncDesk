# Contributing to the Project

First off, thank you for considering contributing! Following these guidelines helps us maintain a high standard of code quality and ensures a smooth development process for everyone.

---

## 1. Branching Strategy

To maintain project integrity, **never push commits directly to the `main` branch.** We follow a structured workflow to keep our development organized:

- **`main` branch**: Represents the latest stable production code. Updates here are only made during a release and are always marked with a **Git Tag** following [Semantic Versioning](https://semver.org/) (e.g., `v1.0.0`).
- **`dev` branch**: This is our primary integration branch. All new features and fixes are merged here first to be tested before a stable release.
- **Topic branches**: Always create a new branch for your work from the `dev` branch. Use the following naming conventions:
  - `feature/<feature-name>` for new features.
  - `fix/<bug-name>` for bug fixes.
  - `docs/<topic>` for documentation updates.

## 2. Commit Message Guidelines

We use a structured commit format to keep our history readable. We use the scope to reference the GitHub Issue number.

**Format:** `<type>(#<issue-number>): <description>`

### Examples

- `feat(#42): add user authentication flow`
- `fix(#15): resolve null pointer exception in ticket list`

### Commit Types

- **feat**: A new feature for the user.
- **fix**: A bug fix for the user.
- **docs**: Documentation changes only.
- **style**: Formatting, missing semi-colons, etc. (no production code change).
- **refactor**: Refactoring production code (e.g., renaming a variable).
- **test**: Adding or refactoring tests (no production code change).
- **chore**: Updating build tasks, package dependencies, etc.

> **Pro-Tip:** Use GitHub keywords like `Close #42` or `Fixes #42` in your PR description (or commit footer) to automatically close the issue once the code is merged.

### Git Commit Template

To make this easier, we suggest installing a local commit template. You can find a recommended configuration in [this Gist](https://gist.github.com/WesleyGoncalves/c4d6799f26931e478e1b74a05c7f0a5a).

## 3. The Pull Request Process

1. **Create your branch** from `dev`.
2. **Commit your changes** using the guidelines above.
3. **Open a Pull Request (PR)** targeting the `dev` branch.
4. **Peer Review**: All PRs must be reviewed and approved by at least one other team member before merging.
5. **Merge**: Once approved, your changes will be merged into `dev`.

More Examples:

- `feat` or `feature`: (new feature for the user, not a new feature for build script)
- `fix`: (bug fix for the user, not a fix to a build script)
- `docs`: (changes to the documentation)
- `style`: (formatting, missing semi colons, etc; no production code change)
- `refactor`: (refactoring production code, eg. renaming a variable)
- `test`: (adding missing tests, refactoring tests; no production code change)
- `chore`: (updating grunt tasks etc; no production code change)
