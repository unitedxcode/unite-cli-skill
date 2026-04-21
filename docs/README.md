# UNITE.app — `unite` CLI (user guide)

This page is for **anyone** who wants to run [UNITE.app](https://www.unitedx.app) from the terminal—**including people who have never used a command line before**. Your AI assistant (**Cursor**, **Codex**, **Copilot**, **Claude**, ChatGPT, etc.) can follow this together with the [agent skill](../skills/unite-cli/SKILL.md) to walk you through each step. You do **not** need Cursor—any tool that can read those Markdown files works.

---

## What you are installing

**UNITE** is a genomics / diagnostics platform in the browser. The **`unite`** program is the **official command-line interface**: upload files, start analysis runs, download results—the same actions as the website, from your computer.

You need:

1. A **free UNITE account** — sign up at [unitedx.app](https://www.unitedx.app).
2. **Python 3.10 or newer** on your computer ([python.org](https://www.python.org/downloads/) or your system package manager).
3. About **five minutes** for the first-time login (one-time setup).

---

## Storage limits (what you can upload)

UNITE applies the **same caps to every account** in the shipped product today (not tiered in code):

- **Total space for your uploads:** **100 GiB** combined across all files you store in UNITE.
- **Largest single file (website upload):** **100 GiB** per file.

To see how much you are using from the terminal after you are logged in, run:

```bash
unite uploads usage
```

That calls the same quota the website uses. If these numbers ever change in production, they will be updated in the main [UNITE.app](https://github.com/unitedx/UNITE.app) repository first; refresh this guide or the [skill](../skills/unite-cli/SKILL.md) if something looks out of date.

---

## Analysis models (names, versions, reference genomes)

Each **model** (for example UNITE-XGB) has a **default version** and a **reference genome** (for example hg19 or hg38). Older versions may still exist for reproducibility. The table below matches the current **`model_catalog.json`** on UNITE.app `main`.

**If you use an AI assistant:** production can update before this page does. Ask it to use **`unite models list`** (same data as the website’s catalog) or to call the live **`/models/catalog`** API; if that output disagrees with the table here, **trust the live pull**.

| Model | Default version | Default reference | Inputs |
|-------|-----------------|-------------------|--------|
| DELFI-NRLAB | 2025.01.03.hg38 | hg38 | BAM |
| FILE-SIZE | 2026.03.30 | — | BAM |
| FRAGLE | 2024.12.20.hg38 | hg38 | BAM |
| FrEIA | 2024.12.18.hg38 | hg38 | BAM |
| LIONHEART | 2024.12.12.hg38 | hg38 | BAM |
| Mega-learner | 2025.01.12.hg38 | hg38 | BAM |
| UNITE-CNN | 2026.03.30.hg38 | hg38 | BAM |
| UNITE-LLM | 2025.01.05.hg38 | hg38 | BAM |
| UNITE-XGB | 2026.03.30.hg19 | hg19 | BAM |
| ichorCNA-TF | 2024.12.25.hg38 | hg38 | BAM |

**Authoritative list (every version and per-version reference):**  
[github.com/unitedx/UNITE.app/blob/main/backend/model_catalog.json](https://github.com/unitedx/UNITE.app/blob/main/backend/model_catalog.json)

The [agent skill](../skills/unite-cli/SKILL.md) repeats this table and embeds a **full JSON snapshot** for assistants that need copy-paste accuracy.

---

## Install `unite` on your computer

### Option A — pipx (recommended)

`pipx` keeps `unite` in its own environment so it does not conflict with other Python tools.

```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
# open a new terminal window, then:
pipx install unite-cli
unite --version
```

If `unite-cli` is not on PyPI yet, install **from source** using the instructions in the [UNITE.app](https://github.com/unitedx/UNITE.app) repository (`cli/` folder).

### Option B — pip

```bash
pip install --user unite-cli
unite --version
```

### After install

Run:

```bash
unite --help
```

You should see a list of commands (`login`, `uploads`, `runs`, …). If the command is not found, make sure your terminal’s `PATH` includes pip’s user bin directory (pipx usually tells you what to add).

---

## First-time sign-in (important)

The CLI uses the **same login** as the website. You never type your password into the terminal; you copy a **short-lived token** from the browser.

1. Open [https://www.unitedx.app](https://www.unitedx.app) and **sign in**.
2. Open the CLI login helper page: **[https://www.unitedx.app/cli/auth](https://www.unitedx.app/cli/auth)**  
3. Click **Copy JSON** (best option—it includes refresh so the CLI stays signed in longer).

### Easiest way to finish login (no giant paste in the terminal)

Save the clipboard to a file, for example on your Desktop:

- **macOS:** paste into TextEdit, save as `unite-login.json`.
- **Windows:** paste into Notepad, save as `unite-login.json`.

Then run (change the path to your file):

```bash
unite login --token-file ~/Desktop/unite-login.json
```

### macOS shortcut (one line)

If you already used **Copy JSON** in the browser:

```bash
pbpaste | unite login --no-browser
```

### Check that it worked

```bash
unite whoami
```

You should see your email or user id and `signed_in: true`. After a successful login, the CLI prints a line ending with **`./authenticated`**—that is only a friendly message, not a folder on your disk.

**Sign out later:** `unite logout`

---

## Everyday tasks (examples)

Your assistant can expand these; **full copy-paste recipes** (batch BAMs, pipelines,
CI, and more) live in the [skill file](../skills/unite-cli/SKILL.md) under **“Recipe snippets”**.

- **List files you uploaded:** `unite uploads list`
- **See how much storage you are using:** `unite uploads usage`
- **Upload sequencing reads:** `unite uploads upload your_file.fastq.gz --type fastq_r1`
- **List your runs:** `unite runs list`
- **Download results for a run:** `unite runs download RUN_ID -o ./my-results/`

Replace `RUN_ID` with the id shown in `unite runs list` or on the website.

---

## More examples (batch work, models)

These use **`unite --json …`** so a script or assistant can parse output (`jq` is
optional but handy). On **Windows**, run them in **Git Bash** or **WSL** unless
you translate the loop yourself.

### Folder of BAM files → UNITE-XGB on each sample

1. Log in once (`unite login`, see above).
2. Run this loop (change the folder path):

```bash
for f in /path/to/your/bams/*.bam; do
  uri=$(unite --json uploads upload "$f" --type bam | jq -r .uri)
  rid=$(unite --json runs submit --file "$uri" --models unite-xgb | jq -r .run_id)
  echo "Started run $rid for $f"
done
```

Ask your assistant to **wait and download** when jobs finish (`unite runs wait`
then `unite runs download`)—patterns are in the [skill recipes](../skills/unite-cli/SKILL.md).

### One sample, several models at once

```bash
unite runs submit \
  --file unite://uploads/my-sample.bam \
  --models unite-xgb,unite-cnn,file-size
```

### Check models and versions from the live server

```bash
unite models list
unite models show "UNITE-XGB"
```

If that output disagrees with a table in the docs or skill, **trust these commands**—they read production.

---

## Using an AI assistant safely

- **Do not paste live login tokens** into public forums, shared screenshots, or untrusted chat logs.
- It is OK to run `unite login` **on your own machine** and keep the JSON file **private** on your disk.
- If an assistant suggests deleting data or running **admin** commands, double-check on the website or with your team first.

---

## Something went wrong?

| Symptom | What to try |
|---------|----------------|
| `command not found: unite` | Re-open the terminal; confirm `pipx` / `pip` finished without errors; check `PATH`. |
| `AUTH_REQUIRED` or login fails | Get a fresh **Copy JSON** from `/cli/auth` and run `unite login` again. |
| `503` / `backend_not_configured` | Your region’s cloud may be temporarily unavailable—try again later or check [unitedx.app](https://www.unitedx.app) status. |
| Paste is awkward in the terminal | Always prefer **`unite login --token-file path/to/file.json`** or **`pbpaste \| unite login --no-browser`** on macOS. |

More detail for **developers and automation** (JSON output, exit codes, cloud regions) is in the [SKILL.md](../skills/unite-cli/SKILL.md) reference.

---

## License

- The **text** in this `docs/` folder and the **skill** under `skills/` are under the [MIT license](../LICENSE) in this repository—you may copy them into your own projects.
- The **UNITE.app product**, website, and **`unite` program source code** remain proprietary; see the [UNITE.app](https://github.com/unitedx/UNITE.app) repository for software and contribution terms.

---

## Where this repo fits

- **This repo (`unitedxcode/unite-cli-skill`):** public skill + this user guide—safe to share and fork for any agentic workflow (IDE skills, CLI agents, or chat).
- **Product source:** [github.com/unitedx/UNITE.app](https://github.com/unitedx/UNITE.app) — application, backend, and CLI implementation.
