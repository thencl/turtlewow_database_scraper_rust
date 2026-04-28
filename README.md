# Turtle Scraper — Index

A complete, agent-buildable Rust + Tauri 2 desktop app that crawls
`https://database.turtlecraft.gg/`, parses every item / spell / quest /
NPC / object page, and stores the result in a local SQLite file.

## Files in this folder (read in this order)

1. **[INIT.md](INIT.md)** — the prompt to paste into your Copilot / Opus / GPT‑5.5 chat.
2. **[SKILLS.md](SKILLS.md)** — the skill list the agent must already know.
3. **[API_REFERENCE.md](API_REFERENCE.md)** — endpoint URLs, CSS selectors, rate limits.
4. **[MASTER_PLAN.md](MASTER_PLAN.md)** — 13 numbered steps with the **complete** source for every file.
5. **README.md** — this file.

## What the agent will produce

```
scraper/
├── INIT.md                ← you start here
├── SKILLS.md
├── API_REFERENCE.md
├── MASTER_PLAN.md
├── README.md
└── turtle-scraper/        ← agent creates this
    ├── package.json, vite.config.ts, tsconfig.json, index.html, .gitignore
    ├── src/                  (vanilla TS UI)
    │   ├── main.ts
    │   └── styles.css
    └── src-tauri/            (Rust backend)
        ├── Cargo.toml
        ├── tauri.conf.json
        ├── build.rs
        ├── capabilities/default.json
        └── src/
            ├── main.rs
            ├── lib.rs
            ├── types.rs
            ├── db.rs
            ├── parser.rs
            ├── rate.rs
            └── scraper.rs
```

## Tech stack (locked)

- **Rust stable** (edition 2021)
- **Tauri 2.x** (desktop shell, no browser)
- **tokio** + **reqwest** + **scraper** + **governor** (async crawler)
- **rusqlite (bundled)** — single `turtle.db` file, WAL mode
- **Vite + TypeScript**, no UI framework

No Python, no Go, no Docker, no cloud. One `.exe` ships everything.

## Rate limits (defaults)

- 8 concurrent requests
- 12 requests / second globally
- 5 retries with exponential backoff (1, 2, 4, 8, 16 s + jitter)
- Honors `Retry-After` and Cloudflare challenge events
- Daily safety cap: 250 000 requests

Full justification + override rules in [API_REFERENCE.md](API_REFERENCE.md) §2.

## Quick start (for a human, not the agent)

```powershell
# 1. install prerequisites once
winget install Rustlang.Rustup
winget install OpenJS.NodeJS.LTS
winget install Microsoft.VisualStudio.2022.BuildTools  # tick "Desktop development with C++"

# 2. open this folder in VS Code, open Copilot Chat
# 3. paste the prompt from INIT.md
# 4. wait. The agent does Steps 0–13 from MASTER_PLAN.md.
```

## License / etiquette

This tool is for personal archival of a community game database that is at
risk of going offline. Be a good neighbour: do not run multiple instances
against the same IP, do not republish scraped content commercially, and
respect any explicit `robots.txt` change the site adds in the future.
