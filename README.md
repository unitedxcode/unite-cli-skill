# unite-cli-skill

Public **Cursor agent skill** + short entry point for the [UNITE.app](https://www.unitedx.app) **`unite`** command-line tool.

| Read this | If you want… |
|-----------|----------------|
| **[docs/README.md](docs/README.md)** | Step-by-step setup, login, and how to use the CLI with **no coding background** (or to help someone else). |
| **[skills/unite-cli/SKILL.md](skills/unite-cli/SKILL.md)** | The full **agent skill** (commands, flags, residency, exit codes, LLM guidance) — copy into your project’s `.cursor/skills/unite-cli/`. |
| **[LICENSE](LICENSE)** | MIT terms for the **skill text and docs** in this repo (not the proprietary UNITE product binary). |

## Install the skill in Cursor (30 seconds)

From a clone of this repo, in **your** project root:

```bash
mkdir -p .cursor/skills
cp -R skills/unite-cli .cursor/skills/
```

Reopen the project. Agents that load skills can now use **`unite-cli`**.

Repository: `git@github.com:unitedxcode/unite-cli-skill.git`
