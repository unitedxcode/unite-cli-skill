# unite-cli-skill

Public **agent skill** (Markdown + YAML frontmatter) for the [UNITE.app](https://www.unitedx.app) **`unite`** CLI. It is **not tied to Cursor**: use it anywhere an LLM or agent runner can load skills, system instructions, or project context—**Cursor**, **GitHub Copilot / Codex**, **Claude Code**, custom tools, or a normal chat by attaching the files.

| Read this | If you want… |
|-----------|----------------|
| **[docs/README.md](docs/README.md)** | Step-by-step setup, login, and how to use the CLI with **no coding background** (or to help someone else). |
| **[skills/unite-cli/SKILL.md](skills/unite-cli/SKILL.md)** | Full **technical + agent** reference (commands, flags, residency, exit codes). |
| **[LICENSE](LICENSE)** | MIT terms for the **skill text and docs** in this repo (not the proprietary UNITE product binary). |

## Wire it into your agent stack

**Cursor (project skill):** from a clone of this repo, at your app root:

```bash
mkdir -p .cursor/skills
cp -R skills/unite-cli .cursor/skills/
```

**Codex / Copilot agents / Claude Code / other CLIs:** use that product’s documented skills or “instructions” directory, **or** point the session at `skills/unite-cli/SKILL.md` (and optionally `docs/README.md`)—no `.cursor/` path required.

**Any LLM UI:** attach `SKILL.md` so the model follows the same behaviour contract.

Repository: `git@github.com:unitedxcode/unite-cli-skill.git`
