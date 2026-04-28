# Turtle Scraper — Index

A complete, agent-buildable Rust + Tauri 2 desktop app that crawls
`https://database.turtlecraft.gg/`, parses every item / spell / quest /
NPC / object page, and stores the result in a local SQLite file.

## Files in this folder (read in this order)

### Agent bootstrap (start here)

1. **[AGENT_DIRECTIVE.md](AGENT_DIRECTIVE.md)** — mandatory rules every agent/AI must read before taking any action.
2. **[INIT.md](INIT.md)** — the prompt to paste into your Copilot / Opus / GPT-5.5 chat to kick off the build.
3. **[SKILLS.md](SKILLS.md)** — the crate/framework knowledge the agent must possess (pinned versions).
4. **[system_prompt.md](system_prompt.md)** — project-wide system-level instructions loaded at every agent session.
5. **[coding_agent_prompt.md](coding_agent_prompt.md)** — standard prompt for implementation / refactoring agent sessions.

### Planning & architecture

6. **[MASTER_PLAN.md](MASTER_PLAN.md)** — 14 numbered steps with the **complete** source for every file.
7. **[plan.prompt.md](plan.prompt.md)** — prompt template for producing detailed implementation plans or handover docs.
8. **[architecture.md](architecture.md)** — system architecture decisions; update whenever a significant decision is made.
9. **[requirements.md](requirements.md)** — all functional and non-functional requirements; source of truth for scope.
10. **[feature_file_correlations.md](feature_file_correlations.md)** — maps every feature area to the source files that implement it.

### Reference

11. **[API_REFERENCE.md](API_REFERENCE.md)** — endpoint URLs, CSS selectors, rate limits, Cloudflare handling.
12. **[security.md](security.md)** — security requirements, standards, and review checklist.
13. **[accessibility.md](accessibility.md)** — accessibility standards and audit process for the UI.
14. **[testing.md](testing.md)** — testing strategy, tooling, and acceptance criteria.

### Tracking

15. **[CHANGELOG.md](CHANGELOG.md)** — append-only record of all project changes.
16. **[checklist.md](checklist.md)** — mandatory quality-gate checklists (pre-commit, pre-release, etc.).
17. **README.md** — this file.

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

## App features (Step 14 premium UI)

| Tab | What it does |
|---|---|
| **Scrape** | Configure kinds, ID range, concurrency, rate; Start / Pause / Resume / Cancel; live progress bar + 8-stat grid (done, found, skipped, errors, req/s, ETA, elapsed, current ID); animated RPS sparkline; session log; Cloudflare challenge banner |
| **Browse** | Search by name, filter by kind, look up exact ID; results grid with quality colours; detail panel showing every structured field + original tooltip HTML |
| **Export** | JSON (re-importable) · CSV · SQL INSERT statements · raw `.db` file copy — all filterable by kind |
| **Import** | Merge a previously exported JSON archive; "Find gaps" scanner to detect missing IDs; one-click "Scrape missing now" targeted run |
| **Profiles** | Save / load / delete named scrape configurations |
| **Stats** | Total record count, per-kind bar chart, skipped counts, DB path / size / dates, clear-skipped button |

Keyboard shortcuts: **Ctrl+1–6** switch tabs · **Ctrl+F** focuses search. Dark/light theme toggle in the top bar.

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
# 4. wait. The agent does Steps 0–14 from MASTER_PLAN.md.
```

## License / etiquette

This tool is for personal archival of a community game database that is at
risk of going offline. Be a good neighbour: do not run multiple instances
against the same IP, do not republish scraped content commercially, and
respect any explicit `robots.txt` change the site adds in the future.
