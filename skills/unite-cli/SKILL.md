---
name: unite-cli
description: >
  Complete reference for the UNITE.app unite-cli (Typer): install, auth, every
  command group, flags, JSON mode, residency, uploads, admin, exit codes,
  troubleshooting, and how LLM/agents should guide users—including zero coding
  experience—safely and step-by-step. Works with Cursor, Codex, Claude Code, or
  any agent that can load this SKILL as context or via a skills directory.
---

# UNITE.app — `unite-cli` skill

Use this skill whenever the user or an agent needs to **install, configure, run,
or debug** the [`unite`](https://www.unitedx.app) command-line interface, or when
an **LLM / agent** should help someone use UNITE from the terminal **without
assuming programming background**.

**New here?** Read the plain-language guide:  
[docs/README.md](https://github.com/unitedxcode/unite-cli-skill/blob/main/docs/README.md) (same repo as the public **MIT** skill bundle: [unitedxcode/unite-cli-skill](https://github.com/unitedxcode/unite-cli-skill)).

**Using this skill (any agentic tool):** the MIT bundle lives at  
`skills/unite-cli/SKILL.md` in [unitedxcode/unite-cli-skill](https://github.com/unitedxcode/unite-cli-skill).  
- **Cursor:** copy `skills/unite-cli/` → `.cursor/skills/unite-cli/` in your project.  
- **Codex CLI, Copilot agents, Claude Code, custom runners:** follow that product’s
  “skills” or project-instructions path, **or** simply attach / `@`-include
  `SKILL.md` in a session—no Cursor-specific layout is required.  
- **Plain chat UIs:** paste or upload `SKILL.md` (and optionally `docs/README.md`)
  so the model has the full contract.

**Inside the UNITE.app monorepo only:** developer docs live at `cli/README.md`;
source under `cli/`; releases via `.github/workflows/release-cli.yml` (`cli-v*` tags).

---

## What the CLI is

- **Name:** `unite` (installed from package `unite-cli`; shared lib `unite-common`).
- **Purpose:** Same capabilities as the web app for uploads, runs, downloads,
  models, demos, workflows—plus **admin-only** ops when a server admin token is
  configured.
- **API path:** By default all HTTP goes to the **Vercel proxy** at
  `https://www.unitedx.app/api/backend/...` (override with `--base-url` or
  `UNITE_BASE_URL`).
- **Auth:** Supabase **access + refresh** JWTs, same session as the website.
  Stored in the **OS keyring** by default; falls back to a `0600` file under
  the config dir if `UNITE_NO_KEYRING=1` or no keyring.

---

## Install (pick one)

| Method | Command | Notes |
|--------|---------|--------|
| **pipx** (recommended) | `pipx install unite-cli` | Isolated env; best for end users. |
| **pip** | `pip install --user unite-cli` | User-wide install. |
| **Homebrew** | `brew install unitedx/unite/unite-cli` | Requires tap `unitedx/unite` to exist and be public. |
| **From source (dev)** | `pip install -e ./common/unite_common && pip install -e './cli[dev]'` | From repo root; always latest code. |

- **Python:** CPython **3.10+** (3.12 / 3.13 recommended).
- **Verify:** `unite --version` and `unite --help`.

If PyPI/Homebrew are not live yet, **source install** is the supported path until
`cli-v*` is pushed and publishing succeeds.

---

## First-time login (critical for agents helping non-coders)

### What the user must have first

1. A **UNITE.app account** and ability to sign in at `https://www.unitedx.app`.
2. **Python + pipx** (or pip) installed; terminal app they can paste into.
3. Understanding that **tokens are secrets** — never paste JWTs into random
   chats, screenshots shared publicly, or agent logs the user does not control.

### Where tokens come from

1. Sign in on the website.
2. Open **`https://www.unitedx.app/cli/auth`** (the CLI can open this URL).
3. Use **Copy JSON** on that page (preferred: includes `access_token`,
   `refresh_token`, and metadata).

### How to run `unite login` (easiest → hardest)

| Situation | Command |
|-----------|---------|
| **Avoid terminal paste** (best for long JSON) | Save clipboard to e.g. `~/Downloads/unite-login.json`, then: `unite login --token-file ~/Downloads/unite-login.json` |
| **macOS clipboard → CLI** | Pipe clipboard into login (see fenced command below). |
| **Interactive** | `unite login` → paste JSON → **Ctrl-D** (macOS/Linux) or **Ctrl-Z** then **Enter** (Windows cmd) to end stdin |
| **Skip opening browser** | Add `--no-browser` (e.g. when using `--token-file` or pipe) |

macOS one-liner (pipe, not a Markdown table cell):

```bash
pbpaste | unite login --no-browser
```

**Flags:**

- `--token-file` / `-f PATH` — read JWT/JSON from file; `-` means read stdin to EOF.
- `--no-browser` — do not open `/cli/auth`.
- `--base-url URL` — non-default site (staging, etc.).
- `--profile NAME` — which profile to store tokens under.

**Accepted paste shapes:**

- Full JSON from **Copy JSON** (includes `access_token`, `refresh_token`, …).
- Bare **access JWT** only — works but **no refresh**; session dies when access
  token expires.

**After success** the CLI prints a green check and text like:
`Successfully logged in as <email>. ./authenticated` (the `./authenticated`
suffix is a friendly status line, not a path on disk).

**Verify:** `unite whoami` — should show `signed_in: true`, profile, base URL.

**Logout:** `unite logout`

---

## Global flags (on every command)

These are registered on the root callback (`unite --help`):

| Flag | Env | Effect |
|------|-----|--------|
| `--json` | — | Machine-readable JSON on stdout; tables/spinners suppressed; errors as structured JSON on stderr. |
| `--no-color` | — | Disable ANSI colors. |
| `-v` / `--verbose` | — | More diagnostic output (repeatable if supported). |
| `--profile NAME` | `UNITE_PROFILE` | Non-default profile (`config.toml`). |
| `--base-url URL` | `UNITE_BASE_URL` | Override proxy base (must match where user signed in). |

**Admin token (not stored in website session):** `UNITE_ADMIN_TOKEN` or profile
`admin_token` in config — required for `unite admin *` only.

---

## Configuration & profiles

- **Config file:** `~/.config/unite/config.toml` (or `$XDG_CONFIG_HOME/unite/`;
  overridden by `UNITE_CONFIG_DIR` in tests).
- **Precedence:** defaults → config file → env → CLI flags.
- **Credentials:** OS keyring service `unite-cli` / account = profile name;
  fallback file `credentials.json.<profile>` under config dir if keyring fails
  or `UNITE_NO_KEYRING=1`.
- **State dir** (upload resume sidecars): `UNITE_STATE_DIR` or platform user
  state dir.

**Multi-environment example:**

```bash
unite --profile staging login --base-url https://staging.example.com
unite --profile staging whoami
```

---

## Command tree (reference)

Root Typer app: `unite_cli.main:app`.

### Top-level

```text
unite login [--token-file|-f PATH] [--no-browser] [--base-url URL] [--profile NAME]
unite logout [--profile NAME]
unite whoami [--profile NAME]
```

### `unite uploads`

```text
list [--profile] [--base-url]
usage
upload <path> [--type ...] [--resumable / --no-resumable] ...
delete <name>
download <name> [-o PATH]
detect-ref <name>
```

Large files on **GCP** use **resumable** uploads by default when over threshold;
see **Resumable uploads** in the UNITE source tree (`cli/README.md`). Sidecar: `.<file>.unite-upload-state`.

### `unite runs`

```text
submit              # single-model / model_inference style
pipeline-submit     # multi-step pipeline with --param KEY=VAL
list
status <run-id>
params <run-id>
score <run-id>
rerun <run-id>
artifacts <run-id>
download <run-id> [-o PATH]
wait <run-id>       # poll until terminal; Ctrl-C keeps run server-side
```

**File references** in run params often use **`unite://uploads/<object>`** URIs
after listing uploads.

### `unite models`

```text
list
show <model-id>
```

### `unite workflows`

```text
list
```

### `unite demo`

```text
list
download <name> [-o PATH]
```

### `unite cloud`

```text
get                 # which lane the proxy will use for this JWT/session
set <gcp|gcp-uk|aliyun>   # **admin only** — sets cookie preference for ops
```

**Non-admin users:** lane is **server-decided** from JWT; CLI cannot override
for normal accounts (data residency).

### `unite admin` (requires admin token)

Mounted sub-apps:

```text
unite admin health
unite admin demo list | upload | delete ...
unite admin models list | upsert | delete ...
unite admin uploads purge-user <user-id>   # destructive; confirmation unless --yes
unite admin archives list | backfill ...
```

Header sent: **`X-Admin-Token`** (from env or profile). All admin routes are
**server-role-gated**; wrong token → 403 / CLI exit **4**.

---

## Exit codes (for agents & CI)

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Generic failure |
| 2 | Invalid CLI usage (Typer) |
| 3 | Auth required — `unite login` |
| 4 | Permission denied (role / admin token) |
| 5 | Residency / cloud mismatch |
| 6 | Transient network/backend after retries |

Prefer **`unite ... --json`** when an agent parses output.

---

## Cloud lanes (high level)

| Lane | Typical users |
|------|----------------|
| `gcp` | Default US-centric |
| `gcp-uk` | EU/UK (GDPR) |
| `aliyun` | Mainland China |

**Rule:** data and compute stay in the same lane; CLI does not move data across
clouds. If `503 backend_not_configured`, that lane’s backend may be down or not
wired in Vercel env — see `unite cloud get` and repo infra docs.

---

## Troubleshooting (quick index)

- **503 `backend_not_configured`:** lane not configured at proxy; not always a
  user bug.
- **401 after login:** expired access token — re-login with fresh JSON from
  `/cli/auth`.
- **Resumable upload stuck:** remove `*.unite-upload-state` and re-upload if
  local file changed.
- **Linux keyring missing:** `export UNITE_NO_KEYRING=1` then `unite login`.
- **Homebrew tap not found:** tap repo may not exist yet; use pipx or source.

Full narrative for developers: in **UNITE.app**, `cli/README.md` → **Troubleshooting**; end users see the same tips in [docs/README.md](https://github.com/unitedxcode/unite-cli-skill/blob/main/docs/README.md).

---

## Releasing the CLI (maintainers)

1. Bump **`version`** in `cli/pyproject.toml` (and align `unite-common` if needed).
2. Tag: `git tag cli-vX.Y.Z && git push origin cli-vX.Y.Z`
3. Workflow `release-cli.yml` runs tests, builds wheels, publishes to PyPI
   (Trusted Publishing), GitHub Release, optional Homebrew bump.

Tag **must** match `cli/pyproject.toml` version per workflow gates.

---

## Guidance for LLM / agentic assistants

### Audience: zero coding experience

1. **One step at a time** — install → verify → open browser page → login →
   `whoami` → then the user’s real task (upload, run, download).
2. **Define terms once:** “terminal”, “paste”, “press Enter”, “JSON file”,
   “home folder (`~`)”.
3. **Prefer file-based login** for long tokens: “Save the file as
   `unite-login.json` on your Desktop, then run exactly this command…”
4. **Never ask the user to paste live JWTs into the agent chat** if the session
   could be logged; prefer instructions to run commands **locally** or use
   `--token-file` from a file **on their machine only**.
5. **Confirm destructive actions** in plain language before suggesting
   `delete`, `purge-user`, or `admin demo delete`.
6. **Use `--json`** when the agent must parse lists (runs, uploads) reliably.

### Audience: developers / CI

- Pin scripts to **`--json`** and exit codes **3 / 5 / 6** for retry vs fail-fast.
- Set **`UNITE_CONFIG_DIR`** in CI with ephemeral dirs + `UNITE_NO_KEYRING=1`.

### When to defer to humans

- Account billing, org policy, **revoking** sessions org-wide.
- **Creating** PyPI projects, Homebrew taps, or rotating **admin** tokens
  (cross-system secrets per repo rules).

---

## Repo map (for implementers)

| Path | Role |
|------|------|
| `cli/src/unite_cli/main.py` | Typer root, global callback |
| `cli/src/unite_cli/commands/auth_cmd.py` | `login` / `logout` / `whoami` |
| `cli/src/unite_cli/client.py` | httpx wrapper, retries, errors |
| `cli/src/unite_cli/auth.py` | Token parse, keyring, JWT decode (unverified exp/sub) |
| `cli/src/unite_cli/config.py` | Profiles, paths, `backend_path()` |
| `frontend/app/cli/auth/page.tsx` | Browser **Copy JSON** source page |

---

## License

- **This skill markdown** is published under **MIT** in
  [unitedxcode/unite-cli-skill](https://github.com/unitedxcode/unite-cli-skill)
  (`skills/unite-cli/SKILL.md`); copy or reference that file in whatever skills
  or instructions layout your agent stack uses (Cursor’s `.cursor/skills/` is
  one example, not a requirement).
- **User-facing prose** for non-coders lives in that repo’s `docs/README.md`.
- **UNITE.app product and `unite-cli` implementation** remain proprietary; see
  the UNITE.app repository’s license terms.
