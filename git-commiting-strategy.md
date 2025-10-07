# Git Committing Strategy — Conventional Commits Specification Guide

### Overview
This guide defines the commit message conventions used in this project.  
It follows the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification to ensure consistent commit messages, semantic versioning, and clear communication of code changes.

---

## Commit Message Format
Every commit message must follow this structured format:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Components
- **type** → The category of change (e.g., `feat`, `fix`, `docs`)
- **scope** → The specific area of the codebase affected (e.g., `auth`, `api`)
- **description** → A short summary of the change
- **body** → (Optional) A detailed explanation of the change
- **footer** → (Optional) References, breaking changes, or metadata (e.g., issue links)

---

## Primary Commit Types
| Type | Purpose | Version Impact |
|------|----------|----------------|
| **feat(scope)** | Introduces a new feature | **MINOR** |
| **fix(scope)** | Fixes a bug | **PATCH** |
| **BREAKING CHANGE** | Introduces incompatible API changes | **MAJOR** |

---

## Additional Commit Types
| Type | Description |
|------|--------------|
| **build(scope)** | Changes to the build system or external dependencies |
| **chore(scope)** | Maintenance tasks that don’t affect source or tests |
| **ci(scope)** | Continuous integration or deployment configuration changes |
| **docs(scope)** | Documentation-only changes |
| **style(scope)** | Code style or formatting changes (no logic changes) |
| **refactor(scope)** | Code restructuring without behavior change |
| **perf(scope)** | Performance optimizations |
| **test(scope)** | Adding or updating tests |

---

## Breaking Changes Syntax
Breaking changes can be declared in two ways:

1. **Using an exclamation mark (`!`) before the colon**
   ```bash
   feat(auth)!: change authentication API
   ```

2. **Using a `BREAKING CHANGE` footer**
   ```bash
   feat(auth): change authentication API

   BREAKING CHANGE: new authentication flow incompatible with v1
   ```

---

## Scope Guidelines
1. **Scope is mandatory** for all commits.  
2. Scopes should be **short, lowercase nouns** describing the affected part of the codebase.  
3. Common scopes include:
   - Feature areas: `auth`, `ui`, `api`, `db`
   - Technical areas: `deps`, `config`, `tests`
   - Components: `button`, `modal`, `form`
   - Services: `auth-service`, `cache-service`
4. Multiple scopes are allowed using commas:
   ```bash
   feat(api,auth): add OAuth2.0 endpoints and authentication flow
   ```

---

## Commit Examples

### Feature Commits
```bash
feat(auth): implement user authentication
feat(ui): add new dashboard components
feat(api)!: redesign public API
```

### Fix Commits
```bash
fix(worker): resolve memory leak
fix(auth): handle expired token refresh
fix(db): resolve connection timeout
```

### Documentation Changes
```bash
docs(readme): update installation steps
docs(api): add JSDoc comments
docs(changelog): add missing entries
```

### Chore and Maintenance
```bash
chore(deps): update dependencies
chore(release): prepare 1.0.0
chore(ci): clean up old workflows
```

### Style Changes
```bash
style(global): format with prettier
style(components): organize imports
```

### Refactoring
```bash
refactor(utils): simplify helper functions
refactor(state): migrate to Redux Toolkit
```

### Performance
```bash
perf(api): implement request caching
perf(db): add query indexes
```

### Testing
```bash
test(auth): add unit tests for auth service
test(e2e): add checkout flow tests
```

### CI Related
```bash
ci(actions): setup GitHub Actions
ci(tests): parallelize test runs
```

### Build System
```bash
build(webpack): add configuration
build(deps): update to React 18
```

### Reverts
```bash
revert(api): feat: add new endpoint

This reverts commit 1234567890abcdef

Reason: Unexpected performance issues in production
```

---

## Breaking Change Example
```bash
feat(vue)!: migrate to Vue 3

This change includes a complete rewrite of the component system
to leverage Vue 3's Composition API and TypeScript support.

BREAKING CHANGE: Components using the Vue 2 API must be rewritten.
Reviewed-by: @techLead
Refs: #123, #456
```

---

## Best Practices
1. **Always include a scope.**
2. Use consistent casing in types and scopes.
3. Separate different changes into different commits.
4. Keep descriptions short, clear, and specific.
5. Include a body when context or rationale is important.
6. Clearly mark breaking changes.
7. Use appropriate commit types for the nature of your change.
8. Reference issues or reviewers in the footer when applicable.

---

## Benefits
✅ Automated changelog generation  
✅ Semantic versioning (MAJOR.MINOR.PATCH)  
✅ Clear change communication  
✅ Easier CI/CD integration  
✅ Simplified code reviews  
✅ Structured project history  
✅ Improved team collaboration  

---

### Reference
📘 **Specification:** [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)  
📂 **File Origin:** `.cursor` rule — Git Usage Policy
