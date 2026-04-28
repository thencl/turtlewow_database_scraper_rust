# SKILLS — What the Agent Must Know

The agent is expected to be fluent in every item below. If a section is unclear, the agent should consult the linked docs **before** writing code, not after.

---

## 1. Rust (edition 2021, stable toolchain)

- Cargo workspaces, features, profiles (`[profile.release]` with `lto = "thin"`, `codegen-units = 1`).
- `tokio` 1.x: `#[tokio::main]`, `spawn`, `JoinSet`, `Semaphore`, `Mutex`, `mpsc`, `broadcast`.
- `reqwest` 0.12 (async, cookie store, gzip, brotli, custom headers, `ClientBuilder`).
- `scraper` 0.20 (CSS selectors via `Selector::parse`, `ElementRef::text`, `inner_html`).
- `serde` + `serde_json` derive + custom `Serialize`/`Deserialize`.
- `anyhow::Result`, `thiserror` for typed errors.
- `tracing` + `tracing-subscriber` for structured logs.
- `governor` 0.6 for token-bucket rate limiting.
- `rusqlite` 0.31 with the `bundled` feature (no system SQLite needed).

Docs: <https://docs.rs/tokio>, <https://docs.rs/reqwest>, <https://docs.rs/scraper>, <https://docs.rs/rusqlite>, <https://docs.rs/governor>.

## 2. Tauri 2.x

- `tauri = "2"` + `tauri-build = "2"`.
- `#[tauri::command]` async commands.
- Emitting events to the frontend with `app.emit("progress", payload)`.
- `tauri.conf.json` v2 schema (security `csp`, `windows[0]`, `bundle.targets`).
- Plugins used: `tauri-plugin-shell`, `tauri-plugin-dialog`, `tauri-plugin-fs` (for export buttons).

Docs: <https://v2.tauri.app/>.

## 3. Frontend (zero-framework, Vite + TypeScript)

- Vite 5, TypeScript 5, no React/Vue. Plain `<script type="module">` and a `main.ts`.
- `@tauri-apps/api/core.invoke` and `@tauri-apps/api/event.listen`.
- CSS Grid for the dashboard. Dark theme, monospace numbers.

## 4. SQLite schema & SQL

- Single file `turtle.db` in the OS app-data dir.
- WAL journal (`PRAGMA journal_mode = WAL;`) for concurrent reads while scraping.
- `INSERT OR REPLACE` keyed on `entry`.
- Indices on `name`, `quality`, `category`.

## 5. Web scraping etiquette

- Identify with a real-looking but truthful UA: `TurtleScraper/1.0 (+local archive; contact: <user>)`.
- Honor `Retry-After` on 429.
- Exponential backoff: 1 s → 2 s → 4 s → 8 s → 16 s, max 5 tries.
- Hard cap: **8 concurrent requests, 12 req/s globally** (see `API_REFERENCE.md`).
- Persistent cookie store so Cloudflare's `cf_clearance` cookie is reused.
- If Cloudflare returns a JS challenge (HTTP 403 + `cf-mitigated: challenge`), pause the queue and surface a "Solve in browser" button in the UI that opens the URL in the system browser; user solves once, cookie is captured via `webview` window.

## 6. Turtle WoW database site shape (as of April 2026)

Endpoints used (all GET, all return HTML):

| Kind  | URL pattern                                       | Listing index                          |
|-------|---------------------------------------------------|----------------------------------------|
| Item  | `https://database.turtlecraft.gg/?item=<id>`      | `https://database.turtlecraft.gg/items` |
| Spell | `https://database.turtlecraft.gg/?spell=<id>`     | `https://database.turtlecraft.gg/spells`|
| Quest | `https://database.turtlecraft.gg/?quest=<id>`     | `https://database.turtlecraft.gg/quests`|
| NPC   | `https://database.turtlecraft.gg/?npc=<id>`       | `https://database.turtlecraft.gg/npcs`  |
| Object| `https://database.turtlecraft.gg/?object=<id>`    | `https://database.turtlecraft.gg/objects`|

Selectors used by the parser are listed in `API_REFERENCE.md` §3. They are stable on the AoWoW fork the site runs.

## 7. Git hygiene

- One commit per completed plan step.
- Commit messages: `step <N>: <title>` (e.g. `step 4: implement rate limiter`).
- `.gitignore` includes `target/`, `node_modules/`, `dist/`, `*.db`, `*.db-wal`, `*.db-shm`.

## 8. Testing

- `cargo test` after every Rust step that adds non-trivial logic.
- Frontend smoke test: `npm run build` must succeed before each commit.
- One end-to-end check: scrape items 65000–65010 against a live site, assert 11 rows in SQLite.

## 9. When to stop and ask

The agent **must stop and ask the user** in only these cases:
- Cloudflare presents a CAPTCHA the cookie reuse flow cannot solve.
- A required dependency is missing AND its install would touch system PATH outside the project.
- The user's existing `turtle.db` already has rows and the agent would overwrite them.

Everything else: decide, document the decision in the commit message, move on.
