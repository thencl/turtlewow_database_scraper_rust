# INIT — Starting Prompt for the Agent

> Paste the block below into a fresh Copilot / Opus / GPT‑5.5 chat **inside this `scraper/` folder**.
> The agent must not leave this folder. It is allowed to create one subfolder named `turtle-scraper/`.

---

## 🟢 PASTE THIS PROMPT TO THE AGENT

You are an autonomous coding agent. Your mission is to build, **from scratch**, a single self-contained desktop application called **Turtle Scraper** that:

1. Crawls every item / spell / quest / NPC page on `https://database.turtlecraft.gg/`.
2. Parses the HTML into structured records.
3. Stores the records in a local **SQLite** file (`turtle.db`).
4. Exposes a web UI inside a **Tauri 2.x** window so the user can start/stop/monitor the scrape, search records, and export to JSON/CSV/SQL.
5. Ships as one `.exe` (Windows) — no Python, no Go, no Docker, no external server.

You will work **only inside this folder**:
`h:\turtle wow 1.17 to 1.18 upgrading mission folder\scraper`
You may create exactly one subfolder: `turtle-scraper/`.

### How to operate

1. Read **every** `.md` file in this folder before you start, in this order:
   1. `INIT.md` (this file)
   2. `SKILLS.md`
   3. `API_REFERENCE.md`
   4. `MASTER_PLAN.md`
   5. `README.md`
2. Then execute `MASTER_PLAN.md` **top to bottom**. Every step has a checkbox `- [ ]`. After you finish a step, edit the file and mark it `- [x]`. Never skip a step.
3. Every code block in `MASTER_PLAN.md` is the **complete, final** content of that file. Copy it verbatim — do not summarize, do not abbreviate, do not insert `// ...rest unchanged`.
4. After every step that changes code, run the verification command listed under **✅ Verify** for that step. If it fails, fix it before moving on.
5. If a tool / dependency is missing, install it with the exact command in `MASTER_PLAN.md`. Do not improvise versions.

### Hard rules

- ❌ No Python in the runtime stack. (Python is allowed only if the user explicitly opts in for a one-off helper.)
- ❌ No local Go service. The scraper logic lives **inside the Tauri Rust backend**.
- ❌ No telemetry, no analytics, no cloud calls except to `database.turtlecraft.gg`.
- ✅ Respect the rate limits in `API_REFERENCE.md` — they are non-negotiable.
- ✅ All long-running work runs in `tokio` tasks; the UI must stay responsive.
- ✅ Every public function gets a doc comment. Every error path uses `anyhow::Result`.
- ✅ Commit after every completed step (`git add -A && git commit -m "step N: <title>"`).
- ✅ Smoketest and recursevly audit and fix any flaws aftre each step to ensure nothing breaks, degrades, regresses or bugs.

-✅ never assume and always websearch if unclear and dont rely on old trainingsdata for fixing anything.
### First action

Reply with exactly:

```
READY. Reading MASTER_PLAN.md now.
```

Then open `MASTER_PLAN.md` and start at **Step 0**.

---

## What the user must do before pasting the prompt

1. Install **Rust stable** (`https://rustup.rs`) — `rustc --version` should print ≥ 1.78.
2. Install **Node.js LTS** (`https://nodejs.org`) — `node -v` ≥ 20.
3. Install the Tauri prerequisites for Windows: **Microsoft C++ Build Tools** + **WebView2** (preinstalled on Win10/11 22H2+).
4. Have `git` on PATH.
5. Open VS Code in this folder, open a new Copilot chat, paste the prompt above.

That's it. The agent does the rest.
