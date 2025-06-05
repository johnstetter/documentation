# Developer Guide: Conventional Commits

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification to ensure a consistent commit history that supports semantic versioning, changelogs, and CI/CD automation.

---

## ✍️ Commit Message Format

```
<type>(<optional scope>): <short description>

[optional body]

[optional footer(s)]
```

---

## 🔤 Types

| Type      | Description                                                  |
|-----------|--------------------------------------------------------------|
| `feat`    | A new feature                                                |
| `fix`     | A bug fix                                                    |
| `docs`    | Documentation changes only                                   |
| `style`   | Changes that do not affect meaning (formatting, whitespace) |
| `refactor`| Code change that neither fixes a bug nor adds a feature      |
| `perf`    | Performance improvement                                      |
| `test`    | Adding or correcting tests                                   |
| `ci`      | CI/CD configuration or scripts                               |
| `build`   | Changes that affect the build system or dependencies         |
| `chore`   | Routine tasks, config, or infrastructure updates             |
| `revert`  | Reverts a previous commit                                    |

---

## 🧠 Scopes

Scopes are optional and help group changes within specific areas of a project (e.g., `feat(api):`, `fix(ci):`, etc.).

---

## 🧪 Examples

```
feat(api): add support for custom headers

fix(ci): correct tag push syntax in GitLab

docs: update README with deployment instructions

chore(deps): bump alpine base image to 3.19

refactor(auth): simplify login flow logic
```

---

## ✅ Why Use Conventional Commits?

- Automates changelog generation
- Enables semantic versioning
- Standardizes Git history
- Powers tools like `semantic-release`
- Improves collaboration and code reviews

---

## 🛠 Tooling Support (Optional)

To enforce commit format locally:

1. Install:
    ```bash
    npm install --save-dev @commitlint/{config-conventional,cli} husky
    ```

2. Add config:
    ```js
    // commitlint.config.js
    module.exports = { extends: ['@commitlint/config-conventional'] };
    ```

3. Hook into Git:
    ```bash
    npx husky install
    npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
    ```

---

## 🚦 Commit Discipline Tips

- Make each commit focused and atomic
- Write the message like you're leaving a note for your future self
- Squash trivial commits before merging (e.g., "fix typo")

---

## 📄 Summary

Use this format for all commits:

```
<type>(<scope>): <description>
```

Be consistent. Be clear. Be kind to your future self and collaborators.

