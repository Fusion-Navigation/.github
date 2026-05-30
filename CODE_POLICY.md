# FNAV Code Policy

Applies to every repository under the Fusion-Navigation GitHub organization.
All contributors are expected to follow these rules without exception.

---

## 1. Repository Map

| Repository | Team | Purpose |
|---|---|---|
| `fnav-core` | core | Shared interfaces, EKF, nav state models — **RPi-only code** |
| `fnav-lab` | core | Navigation pipeline R&D: algorithms, visualizations, data processing, and tests |
| `fnav-solar` | sensors | Polarization compass module |
| `fnav-stellar` | sensors | Celestial navigation: CNN constellation classifier + star-based position estimation |
| `fnav-data-capture` | mobile | Mobile sensor capture app |
| `fnav-ui` | frontend | Ground station / demo UI |

### `fnav-core` rule

`fnav-core` contains **only what runs on the RPi**: shared interfaces, data models, and the EKF core.
No tests, no scripts, no simulation utilities, no tooling — nothing that is not deployed to hardware.
Tests for core logic live in `fnav-lab`.

### Module distribution

Each module (`fnav-solar`, `fnav-stellar`) is published as a private Python
package to GitHub Packages. `fnav-core` imports them as versioned dependencies. Module repos
never import from each other directly.

---

## 2. Branching Strategy

```
main          ← production-ready, protected — no direct push, ever
dev           ← integration branch
feature/*     ← new work, branched off dev
fix/*         ← bug fixes, branched off dev (or main for hotfixes)
release/*     ← release candidates, cut from dev, merged to main
```

Rules:
- Branch names: lowercase, hyphens only — `feature/rayleigh-coarse-search`, `fix/ekf-div-zero`
- Delete branches immediately after merge
- Never rebase or force-push to `main` or `dev`

---

## 3. Pull Requests

### When to open
Every change goes through a PR — no exceptions, including small fixes.
One logical change per PR. Keep scope tight.

### Large PRs
A PR is considered large if it touches more than ~500 lines or crosses module boundaries.
Large PRs must be discussed and agreed upon before work starts — open a GitHub Issue,
describe the change and its scope, and get required reviewer sign-off on the approach.
PRs opened without prior discussion may be closed and sent back to the issue stage.

### Required to merge

| Target | Approvals | CI |
|---|---|---|
| `main` | required reviewer | all checks green |
| `dev` | required reviewer | all checks green |

- Squash merge only — keeps history readable
- Do not merge your own PR without review
- Draft PRs are allowed for early feedback; mark as ready before requesting review

### PR description must include

1. What changed (1–3 bullets)
2. Why it changed
3. How to verify

---

## 4. Code Style

### Formatter: `ruff format`

```bash
ruff format .
```

Two settings, no exceptions:
- **Double quotes** everywhere
- **120 character** max line length

CI rejects non-compliant code. No manual style decisions — ruff is the authority.

### Linter: `ruff check` (F401, F821, N-rules)

```bash
ruff check .
```

| Rule | What it catches |
|---|---|
| `F401` | unused imports |
| `F821` | undefined names (typos, missing imports) |
| `N` (except N806, N803) | naming violations — classes and functions; N806/N803 ignored to allow uppercase math variables (`K`, `F`, `Rz`, etc.) |

No import order rules enforced.

### Naming

| Thing | Convention |
|---|---|
| Classes | `PascalCase` |
| Functions, variables | `snake_case` |
| Constants | `UPPER_SNAKE_CASE` |

### Type hints

Required on all public functions — anything in `fnav-core` or crossing a module boundary.
Internal helpers: optional.

```python
# correct
def compute_heading(lat: float, lon: float, timestamp_utc: float) -> float | None:
    ...

# wrong
def compute_heading(lat, lon, timestamp_utc):
    ...
```

### Comments

Comments are allowed. Don't use them to compensate for a bad name — rename the thing instead.

---

## 5. Commit Messages

Format: `<type>: <short description>`

| Type | When |
|---|---|
| `feat` | new capability |
| `fix` | bug fix |
| `refactor` | restructure without behavior change |
| `docs` | documentation only |
| `chore` | tooling, deps, CI, build |
| `test` | tests only |

Examples:
```
feat: add parabolic interpolation to Rayleigh heading search
fix: clamp solar elevation to prevent division by zero at sunset
refactor: extract IMU noise model into NavigationModule base
chore: upgrade ruff to 0.5.0
```

Rules:
- Max 72 characters on the subject line
- Lowercase, no trailing period
- Present tense — "add", not "added"

---

## 6. Secrets and Credentials

Zero tolerance. No API keys, passwords, hardware serials, AWS credentials, or key material in code. Ever.

- Use `.env` files locally; `.env` is always in `.gitignore`
- If a secret is accidentally committed: treat it as compromised immediately, rotate it,
  then remove it from history with `git filter-repo`

---

## 7. Module Interface Contract

Every navigation module implements the interface defined in `fnav-core`. This is not optional.
The firmware composes modules through this interface — it does not know module internals.

Adding a new sensor module:
1. Required reviewer creates the repo and grants team access
2. Implement the `NavigationModule` interface from `fnav-core`
3. Publish as a versioned private package to GitHub Packages
4. Required reviewer adds it as a dependency in `fnav-firmware/pyproject.toml`
5. Nothing else in the firmware changes

---

## 8. Dependency Management

- `pyproject.toml` and lock files can only be modified with required reviewer approval — no exceptions
- Lock file must be committed alongside every `pyproject.toml` change
- Do not add a dependency without team discussion

---

## 9. CI Checks (required for all PRs)

All checks run in GitHub Actions on every PR. Nothing is enforced locally.

| Check | Tool | What it catches |
|---|---|---|
| Lint | `ruff check` | unused imports, undefined names, naming violations |
| Format | `ruff format --check` | quotes, line length, formatting drift |
| PR validation | custom checks | description sections present, commit message format, branch name |
| Tests | `pytest` | regressions — runs only if `tests/` exists in the repo |

CI config lives in `.github/workflows/`. Do not modify CI files without required reviewer approval.
