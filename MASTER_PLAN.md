# MASTER PLAN — Turtle Scraper, Built From Scratch

> Follow every step in order. Every code block is **the entire file** at that point — copy verbatim.
> Mark each `- [ ]` to `- [x]` when done. Run the **✅ Verify** command before moving on.

Working directory throughout: `h:\turtle wow 1.17 to 1.18 upgrading mission folder\scraper`
All new code goes into the subfolder `turtle-scraper/`.

---

## Step 0 — Sanity check the host machine

- [ ] In a PowerShell at the working directory run:

```powershell
rustc --version
cargo --version
node --version
npm --version
git --version
```

Required: `rustc ≥ 1.78`, `node ≥ 20`, `npm ≥ 10`, `git ≥ 2.40`.

✅ Verify: every command prints a version (no "not recognized").
If any are missing, install per `INIT.md` and re-run.

---

## Step 1 — Scaffold the Tauri 2 project

- [ ] From `scraper/` run:

```powershell
npm create tauri-app@latest -- --yes --identifier gg.turtlecraft.scraper --template vanilla-ts --manager npm turtle-scraper
cd turtle-scraper
npm install
```

- [ ] Add the Rust dependencies we need (overwrites the generated `Cargo.toml` later anyway):

```powershell
cargo add --manifest-path src-tauri/Cargo.toml `
    tokio --features full `
    reqwest --features "cookies,gzip,brotli,json,rustls-tls" `
    scraper@0.20 `
    serde --features derive `
    serde_json `
    anyhow `
    thiserror `
    tracing tracing-subscriber `
    governor@0.6 `
    rusqlite --features bundled `
    rand `
    futures `
    url@2 `
    once_cell
```

- [ ] Add the Tauri plugins:

```powershell
cargo add --manifest-path src-tauri/Cargo.toml `
    tauri-plugin-shell tauri-plugin-dialog tauri-plugin-fs
npm install @tauri-apps/plugin-shell @tauri-apps/plugin-dialog @tauri-apps/plugin-fs
```

✅ Verify: `cargo check --manifest-path turtle-scraper/src-tauri/Cargo.toml` resolves (the full Tauri dev verify moves to after Step 3 when all config is in final form).

---

## Step 2 — Lock the project files

Replace the generated files with the canonical versions below.

### 2.1 — `turtle-scraper/.gitignore`

```gitignore
# Build artifacts
node_modules/
dist/
src-tauri/target/

# OS / IDE
.DS_Store
Thumbs.db
.vscode/
.idea/

# Runtime data
*.db
*.db-wal
*.db-shm
turtle.db*
logs/
*.log

# NOTE: src-tauri/Cargo.lock is intentionally committed (application, not library)
```

### 2.2 — `turtle-scraper/package.json`

```json
{
  "name": "turtle-scraper",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "tauri": "tauri"
  },
  "dependencies": {
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-dialog": "^2.0.0",
    "@tauri-apps/plugin-fs": "^2.0.0",
    "@tauri-apps/plugin-shell": "^2.0.0"
  },
  "devDependencies": {
    "@tauri-apps/cli": "^2.0.0",
    "typescript": "^5.4.0",
    "vite": "^5.2.0"
  }
}
```

### 2.3 — `turtle-scraper/tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts"]
}
```

### 2.4 — `turtle-scraper/vite.config.ts`

```ts
import { defineConfig } from "vite";

export default defineConfig({
  clearScreen: false,
  server: {
    port: 1420,
    strictPort: true,
    watch: { ignored: ["**/src-tauri/**"] }
  },
  envPrefix: ["VITE_", "TAURI_"],
  build: {
    target: "es2022",
    minify: "esbuild",
    sourcemap: false
  }
});
```

✅ Verify: `npm run build` succeeds (it builds the empty frontend).

---

## Step 3 — Tauri configuration

### 3.1 — `turtle-scraper/src-tauri/tauri.conf.json`

```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "Turtle Scraper",
  "version": "1.0.0",
  "identifier": "gg.turtlecraft.scraper",
  "build": {
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build",
    "devUrl": "http://localhost:1420",
    "frontendDist": "../dist"
  },
  "app": {
    "windows": [
      {
        "title": "Turtle Scraper",
        "width": 1440,
        "height": 900,
        "minWidth": 1024,
        "minHeight": 680,
        "resizable": true,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": "default-src 'self'; img-src 'self' data: https://database.turtlecraft.gg; connect-src 'self' ipc: http://ipc.localhost https://database.turtlecraft.gg; style-src 'self' 'unsafe-inline'; script-src 'self'"
    }
  },
  "bundle": {
    "active": true,
    "targets": ["msi", "nsis"],
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.ico"
    ]
  }
}
```

### 3.2 — `turtle-scraper/src-tauri/capabilities/default.json`

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "Default permissions for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:event:default",
    "core:window:default",
    "shell:default",
    "shell:allow-open",
    "dialog:default",
    "fs:default",
    "fs:allow-write-text-file",
    "fs:allow-write-file"
  ]
}
```

### 3.3 — `turtle-scraper/src-tauri/Cargo.toml`

```toml
[package]
name = "turtle-scraper"
version = "1.0.0"
description = "Turtle WoW database archive tool"
authors = ["you"]
edition = "2021"
rust-version = "1.78"

[lib]
name = "turtle_scraper_lib"
crate-type = ["cdylib", "rlib"]

[build-dependencies]
tauri-build = { version = "2", features = [] }

[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-shell = "2"
tauri-plugin-dialog = "2"
tauri-plugin-fs = "2"

tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", default-features = false, features = ["cookies", "gzip", "brotli", "json", "rustls-tls"] }
scraper = "0.20"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
anyhow = "1"
thiserror = "1"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
governor = "0.6"
rusqlite = { version = "0.31", features = ["bundled"] }
rand = "0.8"
futures = "0.3"
url = "2"
once_cell = "1"

[profile.release]
lto = "thin"
codegen-units = 1
panic = "abort"
strip = true
opt-level = 3
```

### 3.4 — `turtle-scraper/src-tauri/build.rs`

```rust
fn main() {
    tauri_build::build();
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml` resolves.

- [ ] Also init git now (before any commits are referenced later):

```powershell
cd turtle-scraper
git init
git add -A
git commit -m "step 3: project locked — Tauri config, capabilities, Cargo.toml"
cd ..
```

- [ ] `npm run tauri dev` builds and the default window opens. Close it — all config is now in final form.

---

## Step 4 — Rust backend: shared types

### 4.1 — `turtle-scraper/src-tauri/src/types.rs`

```rust
//! Shared data types between the scraper, the DB layer, and the frontend.

use serde::{Deserialize, Serialize};

/// The five entity kinds the site exposes.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Serialize, Deserialize)]
#[serde(rename_all = "lowercase")]
pub enum Kind {
    Item,
    Spell,
    Quest,
    Npc,
    Object,
}

impl Kind {
    pub fn as_str(self) -> &'static str {
        match self {
            Kind::Item => "item",
            Kind::Spell => "spell",
            Kind::Quest => "quest",
            Kind::Npc => "npc",
            Kind::Object => "object",
        }
    }

    pub fn url(self, id: u32) -> String {
        format!(
            "https://database.turtlecraft.gg/?{}={}",
            self.as_str(),
            id
        )
    }

    pub fn all() -> [Kind; 5] {
        [Kind::Item, Kind::Spell, Kind::Quest, Kind::Npc, Kind::Object]
    }
}

/// One scraped record — a flexible JSON blob plus a few indexable columns.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Record {
    pub kind: Kind,
    pub entry: u32,
    pub name: String,
    pub category: Option<String>,
    pub quality: Option<i32>,
    pub fields: serde_json::Value,
    pub tooltip_html: String,
    pub fetched_at: i64, // unix seconds
}

/// Live progress emitted to the frontend.
#[derive(Debug, Clone, Serialize)]
pub struct Progress {
    pub kind: String,
    pub queued: u64,
    pub done: u64,
    pub found: u64,
    pub skipped: u64,
    pub errors: u64,
    pub current_id: u32,
    pub req_per_sec: f64,
    pub state: &'static str, // "idle" | "running" | "paused" | "challenge" | "done" | "error"
}

/// Configuration the user can change from the UI before pressing Start.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ScrapeConfig {
    pub kinds: Vec<Kind>,
    pub start_id: u32,
    pub end_id: u32,
    pub concurrency: u32, // hard-capped at 16
    pub rate_per_sec: u32, // hard-capped at 24
    pub use_sitemap: bool,
}

impl Default for ScrapeConfig {
    fn default() -> Self {
        Self {
            kinds: vec![Kind::Item],
            start_id: 1,
            end_id: 90_000,
            concurrency: 8,
            rate_per_sec: 12,
            use_sitemap: true,
        }
    }
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml`.

---

## Step 5 — SQLite layer

### 5.1 — `turtle-scraper/src-tauri/src/db.rs`

```rust
//! SQLite persistence. Single file `turtle.db` next to the executable
//! (or in the OS app-data dir when bundled).

use anyhow::{Context, Result};
use rusqlite::{params, Connection};
use std::path::{Path, PathBuf};
use std::sync::Mutex;

use crate::types::{Kind, Record};

pub struct Db {
    conn: Mutex<Connection>,
    path: PathBuf,
}

impl Db {
    pub fn open(path: impl AsRef<Path>) -> Result<Self> {
        let path = path.as_ref().to_path_buf();
        if let Some(p) = path.parent() {
            std::fs::create_dir_all(p).ok();
        }
        let conn = Connection::open(&path)
            .with_context(|| format!("opening sqlite at {}", path.display()))?;
        conn.execute_batch(
            r#"
            PRAGMA journal_mode = WAL;
            PRAGMA synchronous = NORMAL;
            PRAGMA foreign_keys = ON;

            CREATE TABLE IF NOT EXISTS records (
                kind        TEXT    NOT NULL,
                entry       INTEGER NOT NULL,
                name        TEXT    NOT NULL,
                category    TEXT,
                quality     INTEGER,
                fields_json TEXT    NOT NULL,
                tooltip_html TEXT   NOT NULL,
                fetched_at  INTEGER NOT NULL,
                PRIMARY KEY (kind, entry)
            );

            CREATE INDEX IF NOT EXISTS idx_records_name ON records(name);
            CREATE INDEX IF NOT EXISTS idx_records_kind ON records(kind);
            CREATE INDEX IF NOT EXISTS idx_records_quality ON records(quality);

            CREATE TABLE IF NOT EXISTS _meta (
                key   TEXT PRIMARY KEY,
                value TEXT NOT NULL
            );

            CREATE TABLE IF NOT EXISTS _skipped (
                kind  TEXT NOT NULL,
                entry INTEGER NOT NULL,
                reason TEXT NOT NULL,
                at    INTEGER NOT NULL,
                PRIMARY KEY (kind, entry)
            );
            "#,
        )?;
        Ok(Self { conn: Mutex::new(conn), path })
    }

    pub fn path(&self) -> &Path {
        &self.path
    }

    pub fn upsert(&self, rec: &Record) -> Result<()> {
        let conn = self.conn.lock().unwrap();
        conn.execute(
            r#"INSERT INTO records (kind, entry, name, category, quality, fields_json, tooltip_html, fetched_at)
               VALUES (?1, ?2, ?3, ?4, ?5, ?6, ?7, ?8)
               ON CONFLICT(kind, entry) DO UPDATE SET
                   name = excluded.name,
                   category = excluded.category,
                   quality = excluded.quality,
                   fields_json = excluded.fields_json,
                   tooltip_html = excluded.tooltip_html,
                   fetched_at = excluded.fetched_at"#,
            params![
                rec.kind.as_str(),
                rec.entry,
                rec.name,
                rec.category,
                rec.quality,
                serde_json::to_string(&rec.fields)?,
                rec.tooltip_html,
                rec.fetched_at,
            ],
        )?;
        Ok(())
    }

    pub fn mark_skipped(&self, kind: Kind, entry: u32, reason: &str) -> Result<()> {
        let conn = self.conn.lock().unwrap();
        let now = std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_secs() as i64;
        conn.execute(
            r#"INSERT OR REPLACE INTO _skipped (kind, entry, reason, at) VALUES (?1, ?2, ?3, ?4)"#,
            params![kind.as_str(), entry, reason, now],
        )?;
        Ok(())
    }

    pub fn count(&self, kind: Option<Kind>) -> Result<u64> {
        let conn = self.conn.lock().unwrap();
        let n: i64 = match kind {
            Some(k) => conn.query_row(
                "SELECT COUNT(*) FROM records WHERE kind = ?1",
                [k.as_str()],
                |r| r.get(0),
            )?,
            None => conn.query_row("SELECT COUNT(*) FROM records", [], |r| r.get(0))?,
        };
        Ok(n as u64)
    }

    pub fn search(&self, q: &str, limit: u32) -> Result<Vec<Record>> {
        let conn = self.conn.lock().unwrap();
        let mut stmt = conn.prepare(
            r#"SELECT kind, entry, name, category, quality, fields_json, tooltip_html, fetched_at
               FROM records
               WHERE name LIKE ?1
               ORDER BY name COLLATE NOCASE
               LIMIT ?2"#,
        )?;
        let pat = format!("%{}%", q);
        let rows = stmt
            .query_map(params![pat, limit], |row| {
                let kind_s: String = row.get(0)?;
                let kind = match kind_s.as_str() {
                    "item" => Kind::Item,
                    "spell" => Kind::Spell,
                    "quest" => Kind::Quest,
                    "npc" => Kind::Npc,
                    _ => Kind::Object,
                };
                let fields_json: String = row.get(5)?;
                Ok(Record {
                    kind,
                    entry: row.get::<_, i64>(1)? as u32,
                    name: row.get(2)?,
                    category: row.get(3)?,
                    quality: row.get(4)?,
                    fields: serde_json::from_str(&fields_json).unwrap_or(serde_json::Value::Null),
                    tooltip_html: row.get(6)?,
                    fetched_at: row.get(7)?,
                })
            })?
            .filter_map(|r| r.ok())
            .collect();
        Ok(rows)
    }

    pub fn export_json(&self, out: &Path) -> Result<u64> {
        // Collect rows while holding the lock, then release before the file write.
        let rows: Vec<serde_json::Value> = {
            let conn = self.conn.lock().unwrap();
            let mut stmt = conn.prepare(
                "SELECT kind, entry, name, category, quality, fields_json, tooltip_html, fetched_at FROM records",
            )?;
            stmt.query_map([], |row| {
                let fj: String = row.get(5)?;
                Ok(serde_json::json!({
                    "kind": row.get::<_, String>(0)?,
                    "entry": row.get::<_, i64>(1)?,
                    "name": row.get::<_, String>(2)?,
                    "category": row.get::<_, Option<String>>(3)?,
                    "quality": row.get::<_, Option<i32>>(4)?,
                    "fields": serde_json::from_str::<serde_json::Value>(&fj).unwrap_or(serde_json::Value::Null),
                    "tooltip_html": row.get::<_, String>(6)?,
                    "fetched_at": row.get::<_, i64>(7)?,
                }))
            })?
            .filter_map(|r| r.ok())
            .collect()
        }; // lock released here — file write happens without holding the mutex
        let n = rows.len() as u64;
        std::fs::write(out, serde_json::to_vec_pretty(&rows)?)?;
        Ok(n)
    }
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml`.

---

## Step 6 — Rate limiter + HTTP client

### 6.1 — `turtle-scraper/src-tauri/src/rate.rs`

```rust
//! Token-bucket rate limiter wrapping `governor`.

use governor::{
    clock::DefaultClock,
    state::{InMemoryState, NotKeyed},
    Quota, RateLimiter,
};
use std::num::NonZeroU32;
use std::sync::Arc;

pub type Limiter = RateLimiter<NotKeyed, InMemoryState, DefaultClock>;

pub fn build(per_sec: u32) -> Arc<Limiter> {
    let n = NonZeroU32::new(per_sec.max(1)).unwrap();
    let quota = Quota::per_second(n).allow_burst(NonZeroU32::new(per_sec.max(1) * 2).unwrap());
    Arc::new(RateLimiter::direct(quota))
}
```

### 6.2 — `turtle-scraper/src-tauri/src/parser.rs`

```rust
//! HTML → Record. Selectors are listed in API_REFERENCE.md §3.

use anyhow::Result;
use scraper::{Html, Selector};
use serde_json::json;

use crate::types::{Kind, Record};

const NOT_FOUND_SENTINEL: &str = "Tried to view a non-existing";

pub enum ParseOutcome {
    NotFound,
    Ok(Record),
}

pub fn parse(kind: Kind, entry: u32, body: &str) -> Result<ParseOutcome> {
    if body.contains(NOT_FOUND_SENTINEL) {
        return Ok(ParseOutcome::NotFound);
    }
    let doc = Html::parse_document(body);

    let h1_sel = Selector::parse("h1.text").unwrap();
    let name = doc
        .select(&h1_sel)
        .next()
        .map(|el| el.text().collect::<String>().trim().to_string())
        .unwrap_or_default();

    if name.is_empty() {
        return Ok(ParseOutcome::NotFound);
    }

    let cat_sel = Selector::parse("div.text > a").unwrap();
    let category = doc
        .select(&cat_sel)
        .nth(1)
        .map(|el| el.text().collect::<String>().trim().to_string());

    // quality from h1 class q0..q5 (items / quests use this)
    let quality = doc
        .select(&h1_sel)
        .next()
        .and_then(|el| el.value().attr("class").map(|s| s.to_string()))
        .and_then(|c| {
            for tok in c.split_whitespace() {
                if let Some(rest) = tok.strip_prefix('q') {
                    if let Ok(n) = rest.parse::<i32>() {
                        return Some(n);
                    }
                }
            }
            None
        });

    let tooltip_sel = Selector::parse("table.infobox").unwrap();
    let tooltip_html = doc
        .select(&tooltip_sel)
        .next()
        .map(|el| el.html())
        .unwrap_or_default();

    let fields = match kind {
        Kind::Item => extract_kv(&doc, &["Item Level", "Requires Level", "Binds", "Durability"]),
        Kind::Spell => extract_kv(&doc, &["School", "Cast time", "Cooldown", "Range"]),
        Kind::Quest => extract_kv(&doc, &["Level", "Required level", "Side", "Type"]),
        Kind::Npc => extract_kv(&doc, &["Level", "React", "Type", "Faction"]),
        Kind::Object => extract_kv(&doc, &["Type", "Faction"]),
    };

    let fetched_at = std::time::SystemTime::now()
        .duration_since(std::time::UNIX_EPOCH)
        .unwrap()
        .as_secs() as i64;

    Ok(ParseOutcome::Ok(Record {
        kind,
        entry,
        name,
        category,
        quality,
        fields: json!(fields),
        tooltip_html,
        fetched_at,
    }))
}

/// Walks `<th>label</th><td>value</td>` pairs across the page and keeps the requested labels.
fn extract_kv(doc: &Html, labels: &[&str]) -> serde_json::Map<String, serde_json::Value> {
    use serde_json::Value;
    let mut out = serde_json::Map::new();
    let row_sel = Selector::parse("table tr").unwrap();
    let th_sel = Selector::parse("th").unwrap();
    let td_sel = Selector::parse("td").unwrap();
    for tr in doc.select(&row_sel) {
        let label = tr
            .select(&th_sel)
            .next()
            .map(|e| e.text().collect::<String>().trim().to_string());
        let value = tr
            .select(&td_sel)
            .next()
            .map(|e| e.text().collect::<String>().trim().to_string());
        if let (Some(l), Some(v)) = (label, value) {
            if labels.iter().any(|w| l.eq_ignore_ascii_case(w)) {
                out.insert(l, Value::String(v));
            }
        }
    }
    out
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml`.

---

## Step 7 — The crawler engine

### 7.1 — `turtle-scraper/src-tauri/src/scraper.rs`

```rust
//! Async crawler. Honors rate + concurrency limits, retries with backoff,
//! emits `Progress` events to the frontend, persists every parsed record.

use anyhow::{anyhow, Context, Result};
use futures::stream::{FuturesUnordered, StreamExt};
use rand::Rng;
use reqwest::header::{HeaderMap, HeaderValue, ACCEPT, ACCEPT_LANGUAGE, REFERER, USER_AGENT};
use std::sync::atomic::{AtomicBool, AtomicU64, Ordering};
use std::sync::Arc;
use std::time::{Duration, Instant};
use tauri::{AppHandle, Emitter};
use tokio::sync::Semaphore;
use tracing::{info, warn};

use crate::db::Db;
use crate::parser::{parse, ParseOutcome};
use crate::rate::{build as build_limiter, Limiter};
use crate::types::{Kind, Progress, ScrapeConfig};

const HARD_CONCURRENCY_CAP: u32 = 16;
const HARD_RATE_CAP: u32 = 24;
const DAILY_REQUEST_BUDGET: u64 = 250_000;

pub struct Engine {
    pub running: AtomicBool,
    pub paused: AtomicBool,
    pub cancel: AtomicBool,
    pub challenge: AtomicBool,
}

impl Engine {
    pub fn new() -> Arc<Self> {
        Arc::new(Self {
            running: AtomicBool::new(false),
            paused: AtomicBool::new(false),
            cancel: AtomicBool::new(false),
            challenge: AtomicBool::new(false),
        })
    }
}

fn build_client() -> Result<reqwest::Client> {
    let mut headers = HeaderMap::new();
    headers.insert(USER_AGENT, HeaderValue::from_static("TurtleScraper/1.0 (+local archive)"));
    headers.insert(ACCEPT, HeaderValue::from_static("text/html,application/xhtml+xml"));
    headers.insert(ACCEPT_LANGUAGE, HeaderValue::from_static("en-US,en;q=0.9"));
    headers.insert(REFERER, HeaderValue::from_static("https://database.turtlecraft.gg/"));
    Ok(reqwest::Client::builder()
        .default_headers(headers)
        .cookie_store(true)
        .gzip(true)
        .brotli(true)
        .timeout(Duration::from_secs(20))
        .connect_timeout(Duration::from_secs(10))
        .build()?)
}

async fn fetch_with_retry(
    client: &reqwest::Client,
    limiter: &Limiter,
    url: &str,
    engine: &Engine,
) -> Result<(u16, String, Option<String>)> {
    let mut delay_secs: u64 = 1;
    for attempt in 0..5u32 {
        // wait for cancel / pause / challenge
        loop {
            if engine.cancel.load(Ordering::Relaxed) {
                return Err(anyhow!("cancelled"));
            }
            if engine.paused.load(Ordering::Relaxed) || engine.challenge.load(Ordering::Relaxed) {
                tokio::time::sleep(Duration::from_millis(250)).await;
                continue;
            }
            break;
        }
        limiter.until_ready().await;

        let resp = match client.get(url).send().await {
            Ok(r) => r,
            Err(e) => {
                warn!("attempt {attempt} network error: {e}");
                tokio::time::sleep(jittered(delay_secs)).await;
                delay_secs = (delay_secs * 2).min(16);
                continue;
            }
        };
        let status = resp.status().as_u16();
        let cf_mit = resp
            .headers()
            .get("cf-mitigated")
            .and_then(|h| h.to_str().ok())
            .map(|s| s.to_string());

        if status == 200 {
            let body = resp.text().await.context("reading body")?;
            return Ok((status, body, cf_mit));
        }
        if status == 429 {
            let retry_after = resp
                .headers()
                .get("retry-after")
                .and_then(|h| h.to_str().ok())
                .and_then(|s| s.parse::<u64>().ok())
                .unwrap_or(30);
            warn!("429 — sleeping {retry_after}s");
            tokio::time::sleep(Duration::from_secs(retry_after)).await;
            continue;
        }
        if status == 403 && cf_mit.as_deref() == Some("challenge") {
            engine.challenge.store(true, Ordering::Relaxed);
            return Err(anyhow!("cloudflare challenge"));
        }
        if (500..600).contains(&status) {
            tokio::time::sleep(jittered(delay_secs)).await;
            delay_secs = (delay_secs * 2).min(16);
            continue;
        }
        // other 4xx: don't retry
        let body = resp.text().await.unwrap_or_default();
        return Ok((status, body, cf_mit));
    }
    Err(anyhow!("exhausted retries for {url}"))
}

fn jittered(base_secs: u64) -> Duration {
    let mut rng = rand::thread_rng();
    let f: f64 = rng.gen_range(0.8..1.2);
    Duration::from_millis((base_secs as f64 * 1000.0 * f) as u64)
}

pub async fn run(
    app: AppHandle,
    engine: Arc<Engine>,
    db: Arc<Db>,
    cfg: ScrapeConfig,
) -> Result<()> {
    if engine.running.swap(true, Ordering::SeqCst) {
        return Err(anyhow!("already running"));
    }
    engine.cancel.store(false, Ordering::Relaxed);
    engine.paused.store(false, Ordering::Relaxed);
    engine.challenge.store(false, Ordering::Relaxed);

    let concurrency = cfg.concurrency.min(HARD_CONCURRENCY_CAP).max(1);
    let rate_per_sec = cfg.rate_per_sec.min(HARD_RATE_CAP).max(1);
    let limiter = build_limiter(rate_per_sec);
    let sem = Arc::new(Semaphore::new(concurrency as usize));
    let client = build_client()?;

    let done = Arc::new(AtomicU64::new(0));
    let found = Arc::new(AtomicU64::new(0));
    let skipped = Arc::new(AtomicU64::new(0));
    let errors = Arc::new(AtomicU64::new(0));
    let total_queued: u64 =
        cfg.kinds.len() as u64 * (cfg.end_id.saturating_sub(cfg.start_id) as u64 + 1);

    let started = Instant::now();
    let mut futs = FuturesUnordered::new();

    'outer: for &kind in &cfg.kinds {
        for id in cfg.start_id..=cfg.end_id {
            if engine.cancel.load(Ordering::Relaxed) {
                break 'outer;
            }
            if done.load(Ordering::Relaxed) >= DAILY_REQUEST_BUDGET {
                warn!("daily request budget hit");
                break 'outer;
            }
            let permit = sem.clone().acquire_owned().await.unwrap();
            let client = client.clone();
            let limiter = limiter.clone();
            let db = db.clone();
            let app = app.clone();
            let engine = engine.clone();
            let done = done.clone();
            let found = found.clone();
            let skipped = skipped.clone();
            let errors = errors.clone();
            let started = started;

            futs.push(tokio::spawn(async move {
                let _permit = permit;
                let url = kind.url(id);
                match fetch_with_retry(&client, &limiter, &url, &engine).await {
                    Ok((200, body, _)) => match parse(kind, id, &body) {
                        Ok(ParseOutcome::Ok(rec)) => {
                            if let Err(e) = db.upsert(&rec) {
                                warn!("db upsert {kind:?}/{id}: {e}");
                                errors.fetch_add(1, Ordering::Relaxed);
                            } else {
                                found.fetch_add(1, Ordering::Relaxed);
                            }
                        }
                        Ok(ParseOutcome::NotFound) => {
                            db.mark_skipped(kind, id, "not-found").ok();
                            skipped.fetch_add(1, Ordering::Relaxed);
                        }
                        Err(e) => {
                            warn!("parse {kind:?}/{id}: {e}");
                            errors.fetch_add(1, Ordering::Relaxed);
                        }
                    },
                    Ok((status, _, _)) => {
                        db.mark_skipped(kind, id, &format!("http-{status}")).ok();
                        skipped.fetch_add(1, Ordering::Relaxed);
                    }
                    Err(e) => {
                        warn!("fetch {kind:?}/{id}: {e}");
                        errors.fetch_add(1, Ordering::Relaxed);
                    }
                }

                let d = done.fetch_add(1, Ordering::Relaxed) + 1;
                let elapsed = started.elapsed().as_secs_f64().max(0.001);
                let rps = d as f64 / elapsed;
                let _ = app.emit(
                    "progress",
                    Progress {
                        kind: kind.as_str().into(),
                        queued: total_queued,
                        done: d,
                        found: found.load(Ordering::Relaxed),
                        skipped: skipped.load(Ordering::Relaxed),
                        errors: errors.load(Ordering::Relaxed),
                        current_id: id,
                        req_per_sec: rps,
                        state: if engine.cancel.load(Ordering::Relaxed) {
                            "cancelled"
                        } else if engine.challenge.load(Ordering::Relaxed) {
                            "challenge"
                        } else if engine.paused.load(Ordering::Relaxed) {
                            "paused"
                        } else {
                            "running"
                        },
                    },
                );
            }));

            // drain finished tasks opportunistically
            while futs.len() >= concurrency as usize * 4 {
                if futs.next().await.is_none() {
                    break;
                }
            }
        }
    }

    while futs.next().await.is_some() {}

    let final_state = if engine.cancel.load(Ordering::Relaxed) {
        "cancelled"
    } else {
        "done"
    };
    let _ = app.emit(
        "progress",
        Progress {
            kind: "all".into(),
            queued: total_queued,
            done: done.load(Ordering::Relaxed),
            found: found.load(Ordering::Relaxed),
            skipped: skipped.load(Ordering::Relaxed),
            errors: errors.load(Ordering::Relaxed),
            current_id: 0,
            req_per_sec: 0.0,
            state: final_state,
        },
    );

    info!("scrape complete: state={final_state}");
    engine.running.store(false, Ordering::SeqCst);
    Ok(())
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml`.

---

## Step 8 — Wire up Tauri commands

### 8.1 — `turtle-scraper/src-tauri/src/lib.rs`

```rust
//! Tauri entry. Exposes commands the frontend calls via `invoke()`.

mod db;
mod parser;
mod rate;
mod scraper;
mod types;

use std::path::PathBuf;
use std::sync::Arc;
use tauri::{Manager, State};
use tracing_subscriber::EnvFilter;

use crate::db::Db;
use crate::scraper::Engine;
use crate::types::{Record, ScrapeConfig};

pub struct AppState {
    pub db: Arc<Db>,
    pub engine: Arc<Engine>,
}

fn db_path(app: &tauri::AppHandle) -> PathBuf {
    let dir = app
        .path()
        .app_data_dir()
        .unwrap_or_else(|_| PathBuf::from("."));
    std::fs::create_dir_all(&dir).ok();
    dir.join("turtle.db")
}

#[tauri::command]
async fn start_scrape(
    app: tauri::AppHandle,
    state: State<'_, AppState>,
    cfg: ScrapeConfig,
) -> Result<(), String> {
    let engine = state.engine.clone();
    let db = state.db.clone();
    tokio::spawn(async move {
        if let Err(e) = scraper::run(app, engine, db, cfg).await {
            tracing::error!("scrape failed: {e}");
        }
    });
    Ok(())
}

#[tauri::command]
fn pause_scrape(state: State<'_, AppState>) {
    state
        .engine
        .paused
        .store(true, std::sync::atomic::Ordering::Relaxed);
}

#[tauri::command]
fn resume_scrape(state: State<'_, AppState>) {
    state
        .engine
        .paused
        .store(false, std::sync::atomic::Ordering::Relaxed);
    state
        .engine
        .challenge
        .store(false, std::sync::atomic::Ordering::Relaxed);
}

#[tauri::command]
fn cancel_scrape(state: State<'_, AppState>) {
    state
        .engine
        .cancel
        .store(true, std::sync::atomic::Ordering::Relaxed);
}

#[tauri::command]
fn db_stats(state: State<'_, AppState>) -> Result<serde_json::Value, String> {
    let total = state.db.count(None).map_err(|e| e.to_string())?;
    let mut by_kind = serde_json::Map::new();
    for k in crate::types::Kind::all() {
        let n = state.db.count(Some(k)).map_err(|e| e.to_string())?;
        by_kind.insert(k.as_str().to_string(), serde_json::json!(n));
    }
    Ok(serde_json::json!({
        "total": total,
        "by_kind": by_kind,
        "path": state.db.path().to_string_lossy(),
    }))
}

#[tauri::command]
fn search(state: State<'_, AppState>, q: String, limit: u32) -> Result<Vec<Record>, String> {
    state.db.search(&q, limit.min(500)).map_err(|e| e.to_string())
}

#[tauri::command]
fn export_json(state: State<'_, AppState>, path: String) -> Result<u64, String> {
    state
        .db
        .export_json(std::path::Path::new(&path))
        .map_err(|e| e.to_string())
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tracing_subscriber::fmt()
        .with_env_filter(EnvFilter::try_from_default_env().unwrap_or_else(|_| EnvFilter::new("info")))
        .init();

    tauri::Builder::default()
        .plugin(tauri_plugin_shell::init())
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_fs::init())
        .setup(|app| {
            let path = db_path(&app.handle());
            let db = Arc::new(Db::open(&path).expect("open sqlite"));
            app.manage(AppState {
                db,
                engine: Engine::new(),
            });
            Ok(())
        })
        .invoke_handler(tauri::generate_handler![
            start_scrape,
            pause_scrape,
            resume_scrape,
            cancel_scrape,
            db_stats,
            search,
            export_json
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### 8.2 — `turtle-scraper/src-tauri/src/main.rs`

```rust
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

fn main() {
    turtle_scraper_lib::run();
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml` is clean (zero warnings is ideal, but `#[allow(dead_code)]` is fine on `Engine` fields if any).

---

## Step 9 — Frontend

### 9.1 — `turtle-scraper/index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Turtle Scraper</title>
    <link rel="stylesheet" href="/src/styles.css" />
  </head>
  <body>
    <header>
      <h1>🐢 Turtle Scraper</h1>
      <div id="dbstats" class="muted">…</div>
    </header>

    <section class="card">
      <h2>Scrape</h2>
      <form id="cfg" class="grid">
        <label>Kinds
          <select id="kinds" multiple size="5">
            <option value="item" selected>item</option>
            <option value="spell">spell</option>
            <option value="quest">quest</option>
            <option value="npc">npc</option>
            <option value="object">object</option>
          </select>
        </label>
        <label>Start ID <input id="start" type="number" value="1" min="1" /></label>
        <label>End ID <input id="end" type="number" value="90000" min="1" /></label>
        <label>Concurrency <input id="conc" type="number" value="8" min="1" max="16" /></label>
        <label>Rate / sec <input id="rate" type="number" value="12" min="1" max="24" /></label>
      </form>
      <div class="row">
        <button id="start-btn">Start</button>
        <button id="pause-btn" disabled>Pause</button>
        <button id="resume-btn" disabled>Resume</button>
        <button id="cancel-btn" disabled>Cancel</button>
        <button id="export-btn">Export JSON…</button>
      </div>
      <div id="progress" class="progress">
        <div id="bar"></div>
      </div>
      <pre id="status">idle</pre>
    </section>

    <section class="card">
      <h2>Search</h2>
      <div class="row">
        <input id="q" placeholder="search by name…" />
        <button id="search-btn">Search</button>
      </div>
      <div id="results"></div>
    </section>

    <section id="challenge" class="card hidden">
      <h2>Cloudflare challenge detected</h2>
      <p>Open the site in your browser, solve the challenge, then click <b>Resume</b>.</p>
      <button id="open-site">Open database.turtlecraft.gg</button>
    </section>

    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

### 9.2 — `turtle-scraper/src/styles.css`

```css
:root {
  --bg: #0e1014;
  --panel: #161a21;
  --border: #232934;
  --text: #e7ecf3;
  --muted: #8a93a3;
  --accent: #5fd07a;
  --warn: #f0b429;
  --err: #ef5350;
  font-family: ui-sans-serif, system-ui, "Segoe UI", sans-serif;
}
* { box-sizing: border-box; }
html, body { margin: 0; background: var(--bg); color: var(--text); }
header { display: flex; align-items: baseline; gap: 1rem; padding: 1rem 1.5rem; border-bottom: 1px solid var(--border); }
header h1 { margin: 0; font-size: 1.25rem; }
.muted { color: var(--muted); font-size: 0.9rem; }
.card { background: var(--panel); border: 1px solid var(--border); border-radius: 8px; margin: 1rem 1.5rem; padding: 1rem 1.25rem; }
.card h2 { margin: 0 0 .75rem; font-size: 1rem; color: var(--muted); text-transform: uppercase; letter-spacing: 0.05em; }
.grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 0.75rem; }
.grid label { display: flex; flex-direction: column; gap: .25rem; font-size: .85rem; color: var(--muted); }
input, select, button { background: #0b0d12; color: var(--text); border: 1px solid var(--border); border-radius: 4px; padding: .4rem .6rem; font: inherit; }
button { cursor: pointer; }
button:hover:not(:disabled) { border-color: var(--accent); }
button:disabled { opacity: 0.4; cursor: not-allowed; }
.row { display: flex; gap: .5rem; margin-top: .75rem; flex-wrap: wrap; }
.progress { margin-top: .75rem; height: 6px; background: #0b0d12; border-radius: 3px; overflow: hidden; }
#bar { height: 100%; width: 0%; background: var(--accent); transition: width 0.2s; }
pre { margin: .5rem 0 0; font-family: ui-monospace, "Cascadia Mono", monospace; font-size: .85rem; color: var(--muted); white-space: pre-wrap; }
#results { display: grid; gap: .35rem; margin-top: .75rem; max-height: 320px; overflow: auto; }
.result { display: grid; grid-template-columns: 80px 80px 1fr 120px; gap: .75rem; padding: .35rem .5rem; border: 1px solid var(--border); border-radius: 4px; font-size: .9rem; }
.q0 { color: #9d9d9d; } .q1 { color: #fff; } .q2 { color: #1eff00; }
.q3 { color: #0070dd; } .q4 { color: #a335ee; } .q5 { color: #ff8000; }
.hidden { display: none; }
```

### 9.3 — `turtle-scraper/src/main.ts`

```ts
import { invoke } from "@tauri-apps/api/core";
import { listen } from "@tauri-apps/api/event";
import { save } from "@tauri-apps/plugin-dialog";
import { open as openShell } from "@tauri-apps/plugin-shell";

type Kind = "item" | "spell" | "quest" | "npc" | "object";

interface Progress {
  kind: string;
  queued: number;
  done: number;
  found: number;
  skipped: number;
  errors: number;
  current_id: number;
  req_per_sec: number;
  state: string;
}

interface ScrapedRecord {
  kind: Kind;
  entry: number;
  name: string;
  category: string | null;
  quality: number | null;
}

const $ = <T extends HTMLElement = HTMLElement>(id: string): T =>
  document.getElementById(id) as T;

async function refreshStats() {
  const s = (await invoke("db_stats")) as {
    total: number;
    by_kind: Record<string, number>;
    path: string;
  };
  $("dbstats").textContent =
    `${s.total.toLocaleString()} records · ${Object.entries(s.by_kind)
      .map(([k, n]) => `${k}:${n}`)
      .join(" · ")} · ${s.path}`;
}

function readConfig() {
  const kinds: Kind[] = Array.from(
    ($("kinds") as HTMLSelectElement).selectedOptions
  ).map((o) => o.value as Kind);
  return {
    kinds,
    start_id: Number(($("start") as HTMLInputElement).value),
    end_id: Number(($("end") as HTMLInputElement).value),
    concurrency: Number(($("conc") as HTMLInputElement).value),
    rate_per_sec: Number(($("rate") as HTMLInputElement).value),
    use_sitemap: true,
  };
}

function setRunning(running: boolean) {
  ($("start-btn") as HTMLButtonElement).disabled = running;
  ($("pause-btn") as HTMLButtonElement).disabled = !running;
  ($("resume-btn") as HTMLButtonElement).disabled = !running;
  ($("cancel-btn") as HTMLButtonElement).disabled = !running;
}

$("start-btn").addEventListener("click", async () => {
  try {
    setRunning(true);
    await invoke("start_scrape", { cfg: readConfig() });
  } catch (err) {
    setRunning(false);
    $("status").textContent = `Error: ${String(err)}`;
  }
});
$("pause-btn").addEventListener("click", () => invoke("pause_scrape"));
$("resume-btn").addEventListener("click", () => {
  $("challenge").classList.add("hidden");
  invoke("resume_scrape");
});
$("cancel-btn").addEventListener("click", () => invoke("cancel_scrape"));
$("open-site").addEventListener("click", () =>
  openShell("https://database.turtlecraft.gg/")
);

$("export-btn").addEventListener("click", async () => {
  const path = await save({
    defaultPath: "turtle-export.json",
    filters: [{ name: "JSON", extensions: ["json"] }],
  });
  if (!path) return;
  const n = (await invoke("export_json", { path })) as number;
  $("status").textContent = `exported ${n} records to ${path}`;
});

$("search-btn").addEventListener("click", async () => {
  const q = ($("q") as HTMLInputElement).value.trim();
  if (!q) return;
  const rows = (await invoke("search", { q, limit: 200 })) as ScrapedRecord[];
  const html = rows
    .map(
      (r) =>
        `<div class="result"><span class="muted">${r.kind}</span><span>${r.entry}</span><span class="q${
          r.quality ?? 1
        }">${escape(r.name)}</span><span class="muted">${escape(r.category ?? "")}</span></div>`
    )
    .join("");
  $("results").innerHTML = html || `<div class="muted">no results</div>`;
});

function escape(s: string): string {
  return s.replace(/[&<>"']/g, (c) =>
    ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c]!)
  );
}

await listen<Progress>("progress", (e) => {
  const p = e.payload;
  const pct = p.queued > 0 ? (p.done / p.queued) * 100 : 0;
  ($("bar") as HTMLDivElement).style.width = `${pct.toFixed(1)}%`;
  $("status").textContent =
    `${p.state.toUpperCase()}  ${p.done.toLocaleString()}/${p.queued.toLocaleString()}` +
    `  found=${p.found}  skipped=${p.skipped}  errors=${p.errors}` +
    `  rps=${p.req_per_sec.toFixed(2)}  id=${p.current_id}`;
  if (p.state === "challenge") {
    $("challenge").classList.remove("hidden");
  }
  if (p.state === "done" || p.state === "cancelled" || p.state === "error") {
    setRunning(false);
    refreshStats();
  }
});

refreshStats();
```

✅ Verify: `npm run build` is clean.

---

## Step 10 — End-to-end smoke test

- [ ] In `turtle-scraper/` run:

```powershell
npm run tauri dev
```

- [ ] When the window opens, set:
  - Kinds: `item`
  - Start ID: `65000`
  - End ID: `65010`
  - Concurrency: `4`
  - Rate / sec: `4`
- [ ] Click **Start**.

✅ Verify: progress bar moves; final status shows `done`; `dbstats` line shows ≥ 1 record. Click **Search**, type a few letters, results appear.

If 403 + cf-mitigated:challenge fires, click "Open database.turtlecraft.gg", solve once in your default browser, return and click **Resume**. (Cookie handoff to the Tauri client happens automatically because both share the same network identity from Cloudflare's POV after one successful manual visit. If the WAF still blocks, lower `rate_per_sec` to `4` and retry.)

---

## Step 11 — Production build

- [ ] Ensure WiX Toolset (for MSI) and NSIS are installed:

```powershell
winget install WiXToolset.WiXToolset
winget install NSIS.NSIS
# Restart the shell so PATH picks up wix / makensis
```

- [ ] In `turtle-scraper/` run:

```powershell
npm run tauri build
```

✅ Verify: `src-tauri/target/release/bundle/msi/*.msi` and `bundle/nsis/*.exe` exist. Double-click the NSIS installer; the app launches; `db_stats` shows the previously-scraped rows (db lives in `%APPDATA%\gg.turtlecraft.scraper\turtle.db`).

---

## Step 12 — Real-world full run

- [ ] Set Kinds = all five, Start ID = `1`, End ID = `90000`, Concurrency = `8`, Rate = `12`.
- [ ] Click **Start**. Walk away. Expected wall-clock at default rate: ~6–8 hours total for ~270 k page checks (most return "not-found" and are skipped fast).
- [ ] If at any point errors > 1 % of done, click **Pause**, drop Rate to `6`, click **Resume**.

✅ Verify: at completion, `db_stats.total ≥ 30 000` (typical archive size for the 1.18.1 build).

---

## Step 13 — Export & ship

- [ ] Click **Export JSON…** → save as `turtle-1.18.1.json`.
- [ ] Optionally run from a PowerShell:

```powershell
sqlite3 "$env:APPDATA\gg.turtlecraft.scraper\turtle.db" .dump > turtle-1.18.1.sql
```

(SQLite CLI is included with Windows 11; if missing, the app's exported JSON is sufficient.)

- [ ] Commit & tag:

```powershell
git add -A
git commit -m "step 13: shippable build"
git tag v1.0.0
```

You are done.

---

## Step 14 — Premium UI overhaul (replaces Step 9 files entirely)

> This step replaces `index.html`, `styles.css`, and `main.ts` produced in Step 9, and extends
> the Rust backend with new commands. Complete Steps 1–13 first, then apply this step on top.
> After this step the app has: tabbed navigation, live scrape dashboard with ETA / rate graph,
> session log, import JSON to continue / enrich / gap-fill, multi-format export (JSON / CSV / SQL),
> per-record detail view with full tooltip HTML mirroring the original site style, scrape profile
> save/load, rescrape-missing mode, dark/light theme toggle, and keyboard shortcuts.

---

### 14.0 — Extend the Rust backend

New commands required by the premium UI. **Append** these to `src-tauri/src/db.rs` inside `impl Db`, then add the new Tauri commands to `src-tauri/src/lib.rs`, and extend `types.rs`.

#### 14.0.1 — `types.rs` additions

Append to the bottom of `turtle-scraper/src-tauri/src/types.rs`:

```rust
/// A saved scrape profile the user can recall.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ScrapeProfile {
    pub name: String,
    pub cfg: ScrapeConfig,
}

/// Summary returned by `gap_ids` — which (kind, id) pairs have no record and
/// were never skipped (i.e. were never attempted, or had errors).
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct GapEntry {
    pub kind: String,
    pub entry: u32,
}

/// Rich stats for the stats panel.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct RichStats {
    pub total: u64,
    pub by_kind: std::collections::HashMap<String, u64>,
    pub skipped_total: u64,
    pub skipped_by_kind: std::collections::HashMap<String, u64>,
    pub db_size_bytes: u64,
    pub path: String,
    pub oldest_fetch: Option<i64>,
    pub newest_fetch: Option<i64>,
}

/// One entry in the session event log.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct LogEntry {
    pub ts: i64,   // unix ms
    pub level: String, // "info" | "warn" | "error"
    pub msg: String,
}
```

#### 14.0.2 — `db.rs` additions

Append to `impl Db` in `turtle-scraper/src-tauri/src/db.rs`:

```rust
    /// Return all records (or records for one kind) as JSON records.
    /// Used for export to JSON/CSV/SQL.
    pub fn all_records(&self, kind_filter: Option<Kind>) -> Result<Vec<crate::types::Record>> {
        let conn = self.conn.lock().unwrap();
        let sql = if kind_filter.is_some() {
            "SELECT kind, entry, name, category, quality, fields_json, tooltip_html, fetched_at
             FROM records WHERE kind = ?1 ORDER BY entry"
        } else {
            "SELECT kind, entry, name, category, quality, fields_json, tooltip_html, fetched_at
             FROM records ORDER BY kind, entry"
        };
        let mut stmt = conn.prepare(sql)?;
        let param: &[&dyn rusqlite::ToSql] = match &kind_filter {
            Some(k) => &[&k.as_str()],
            None => &[],
        };
        let rows = stmt
            .query_map(rusqlite::params_from_iter(param.iter()), |row| {
                let kind_s: String = row.get(0)?;
                let kind = match kind_s.as_str() {
                    "item" => Kind::Item,
                    "spell" => Kind::Spell,
                    "quest" => Kind::Quest,
                    "npc" => Kind::Npc,
                    _ => Kind::Object,
                };
                let fields_json: String = row.get(5)?;
                Ok(crate::types::Record {
                    kind,
                    entry: row.get::<_, i64>(1)? as u32,
                    name: row.get(2)?,
                    category: row.get(3)?,
                    quality: row.get(4)?,
                    fields: serde_json::from_str(&fields_json).unwrap_or(serde_json::Value::Null),
                    tooltip_html: row.get(6)?,
                    fetched_at: row.get(7)?,
                })
            })?
            .filter_map(|r| r.ok())
            .collect();
        Ok(rows)
    }

    /// Find (kind, entry) pairs in [start_id, end_id] that have neither a record
    /// nor a _skipped entry. These are the "gaps" for a re-scrape-missing run.
    pub fn gap_ids(&self, kinds: &[Kind], start_id: u32, end_id: u32) -> Result<Vec<crate::types::GapEntry>> {
        let conn = self.conn.lock().unwrap();
        let mut gaps = Vec::new();
        for &kind in kinds {
            let ks = kind.as_str();
            let mut stmt = conn.prepare(
                "SELECT entry FROM records WHERE kind = ?1 AND entry >= ?2 AND entry <= ?3",
            )?;
            let present: std::collections::HashSet<u32> = stmt
                .query_map(rusqlite::params![ks, start_id, end_id], |r| {
                    r.get::<_, i64>(0).map(|v| v as u32)
                })?
                .filter_map(|r| r.ok())
                .collect();
            let mut skip_stmt = conn.prepare(
                "SELECT entry FROM _skipped WHERE kind = ?1 AND entry >= ?2 AND entry <= ?3",
            )?;
            let skipped: std::collections::HashSet<u32> = skip_stmt
                .query_map(rusqlite::params![ks, start_id, end_id], |r| {
                    r.get::<_, i64>(0).map(|v| v as u32)
                })?
                .filter_map(|r| r.ok())
                .collect();
            for id in start_id..=end_id {
                if !present.contains(&id) && !skipped.contains(&id) {
                    gaps.push(crate::types::GapEntry { kind: ks.to_string(), entry: id });
                }
            }
        }
        Ok(gaps)
    }

    /// Rich stats.
    pub fn rich_stats(&self) -> Result<crate::types::RichStats> {
        let conn = self.conn.lock().unwrap();
        let total: i64 = conn.query_row("SELECT COUNT(*) FROM records", [], |r| r.get(0))?;
        let skipped_total: i64 =
            conn.query_row("SELECT COUNT(*) FROM _skipped", [], |r| r.get(0))?;
        let mut by_kind = std::collections::HashMap::new();
        let mut skipped_by_kind = std::collections::HashMap::new();
        for k in crate::types::Kind::all() {
            let n: i64 = conn.query_row(
                "SELECT COUNT(*) FROM records WHERE kind = ?1",
                [k.as_str()],
                |r| r.get(0),
            )?;
            let s: i64 = conn.query_row(
                "SELECT COUNT(*) FROM _skipped WHERE kind = ?1",
                [k.as_str()],
                |r| r.get(0),
            )?;
            by_kind.insert(k.as_str().to_string(), n as u64);
            skipped_by_kind.insert(k.as_str().to_string(), s as u64);
        }
        let oldest: Option<i64> =
            conn.query_row("SELECT MIN(fetched_at) FROM records", [], |r| r.get(0)).ok().flatten();
        let newest: Option<i64> =
            conn.query_row("SELECT MAX(fetched_at) FROM records", [], |r| r.get(0)).ok().flatten();
        drop(conn);
        let db_size = std::fs::metadata(&self.path).map(|m| m.len()).unwrap_or(0);
        Ok(crate::types::RichStats {
            total: total as u64,
            by_kind,
            skipped_total: skipped_total as u64,
            skipped_by_kind,
            db_size_bytes: db_size,
            path: self.path.to_string_lossy().into_owned(),
            oldest_fetch: oldest,
            newest_fetch: newest,
        })
    }

    /// Import records from a JSON array (previously exported). Returns (inserted, updated, skipped).
    pub fn import_json(&self, data: &[u8]) -> Result<(u64, u64, u64)> {
        let records: Vec<serde_json::Value> = serde_json::from_slice(data)?;
        let mut inserted = 0u64;
        let mut updated = 0u64;
        let mut skipped_count = 0u64;
        let conn = self.conn.lock().unwrap();
        for v in &records {
            let kind_s = match v["kind"].as_str() {
                Some(s) => s,
                None => { skipped_count += 1; continue; }
            };
            let entry = match v["entry"].as_i64() {
                Some(n) => n,
                None => { skipped_count += 1; continue; }
            };
            let name = v["name"].as_str().unwrap_or("").to_string();
            let category = v["category"].as_str().map(|s| s.to_string());
            let quality = v["quality"].as_i64().map(|n| n as i32);
            let fields = serde_json::to_string(&v["fields"]).unwrap_or_else(|_| "null".into());
            let tooltip = v["tooltip_html"].as_str().unwrap_or("").to_string();
            let fetched_at = v["fetched_at"].as_i64().unwrap_or(0);

            let existing: Option<i64> = conn
                .query_row(
                    "SELECT fetched_at FROM records WHERE kind=?1 AND entry=?2",
                    rusqlite::params![kind_s, entry],
                    |r| r.get(0),
                )
                .ok();
            if existing.is_some() {
                conn.execute(
                    "UPDATE records SET name=?3,category=?4,quality=?5,fields_json=?6,tooltip_html=?7,fetched_at=?8 WHERE kind=?1 AND entry=?2",
                    rusqlite::params![kind_s, entry, name, category, quality, fields, tooltip, fetched_at],
                )?;
                updated += 1;
            } else {
                conn.execute(
                    "INSERT INTO records (kind,entry,name,category,quality,fields_json,tooltip_html,fetched_at) VALUES (?1,?2,?3,?4,?5,?6,?7,?8)",
                    rusqlite::params![kind_s, entry, name, category, quality, fields, tooltip, fetched_at],
                )?;
                inserted += 1;
            }
        }
        Ok((inserted, updated, skipped_count))
    }

    /// Export records as CSV. Returns bytes.
    pub fn export_csv(&self, kind_filter: Option<Kind>) -> Result<Vec<u8>> {
        let recs = self.all_records(kind_filter)?;
        let mut out = Vec::new();
        out.extend_from_slice(b"kind,entry,name,category,quality,fetched_at\n");
        for r in &recs {
            let row = format!(
                "{},{},{},{},{},{}\n",
                r.kind.as_str(),
                r.entry,
                csv_escape(&r.name),
                csv_escape(r.category.as_deref().unwrap_or("")),
                r.quality.map(|q| q.to_string()).unwrap_or_default(),
                r.fetched_at,
            );
            out.extend_from_slice(row.as_bytes());
        }
        Ok(out)
    }

    /// Export records as SQL INSERT statements.
    pub fn export_sql(&self, kind_filter: Option<Kind>) -> Result<Vec<u8>> {
        let recs = self.all_records(kind_filter)?;
        let mut out = Vec::new();
        out.extend_from_slice(b"BEGIN;\n");
        for r in &recs {
            let row = format!(
                "INSERT OR REPLACE INTO records (kind,entry,name,category,quality,fields_json,tooltip_html,fetched_at) VALUES ('{}',{},'{}',{},{},{},'{}',{});\n",
                r.kind.as_str(),
                r.entry,
                sql_escape(&r.name),
                r.category.as_deref().map(|s| format!("'{}'", sql_escape(s))).unwrap_or_else(|| "NULL".into()),
                r.quality.map(|q| q.to_string()).unwrap_or_else(|| "NULL".into()),
                serde_json::to_string(&r.fields).map(|s| format!("'{}'", sql_escape(&s))).unwrap_or_else(|_| "NULL".into()),
                sql_escape(&r.tooltip_html),
                r.fetched_at,
            );
            out.extend_from_slice(row.as_bytes());
        }
        out.extend_from_slice(b"COMMIT;\n");
        Ok(out)
    }

    /// Save / load scrape profiles from _meta table.
    pub fn save_profile(&self, p: &crate::types::ScrapeProfile) -> Result<()> {
        let conn = self.conn.lock().unwrap();
        let key = format!("profile:{}", p.name);
        let val = serde_json::to_string(p)?;
        conn.execute(
            "INSERT OR REPLACE INTO _meta (key, value) VALUES (?1, ?2)",
            rusqlite::params![key, val],
        )?;
        Ok(())
    }

    pub fn load_profiles(&self) -> Result<Vec<crate::types::ScrapeProfile>> {
        let conn = self.conn.lock().unwrap();
        let mut stmt = conn.prepare("SELECT value FROM _meta WHERE key LIKE 'profile:%'")?;
        let profiles = stmt
            .query_map([], |r| r.get::<_, String>(0))?
            .filter_map(|r| r.ok())
            .filter_map(|s| serde_json::from_str::<crate::types::ScrapeProfile>(&s).ok())
            .collect();
        Ok(profiles)
    }

    pub fn delete_profile(&self, name: &str) -> Result<()> {
        let conn = self.conn.lock().unwrap();
        let key = format!("profile:{name}");
        conn.execute("DELETE FROM _meta WHERE key = ?1", rusqlite::params![key])?;
        Ok(())
    }

    /// Get a single record by kind + entry.
    pub fn get_record(&self, kind: Kind, entry: u32) -> Result<Option<crate::types::Record>> {
        let conn = self.conn.lock().unwrap();
        let res = conn.query_row(
            "SELECT kind,entry,name,category,quality,fields_json,tooltip_html,fetched_at FROM records WHERE kind=?1 AND entry=?2",
            rusqlite::params![kind.as_str(), entry],
            |row| {
                let fields_json: String = row.get(5)?;
                Ok(crate::types::Record {
                    kind,
                    entry: row.get::<_, i64>(1)? as u32,
                    name: row.get(2)?,
                    category: row.get(3)?,
                    quality: row.get(4)?,
                    fields: serde_json::from_str(&fields_json).unwrap_or(serde_json::Value::Null),
                    tooltip_html: row.get(6)?,
                    fetched_at: row.get(7)?,
                })
            },
        );
        match res {
            Ok(r) => Ok(Some(r)),
            Err(rusqlite::Error::QueryReturnedNoRows) => Ok(None),
            Err(e) => Err(e.into()),
        }
    }

    /// Clear all _skipped entries for given kinds — allows re-scraping skipped IDs.
    pub fn clear_skipped(&self, kinds: &[Kind]) -> Result<u64> {
        let conn = self.conn.lock().unwrap();
        let mut total = 0u64;
        for k in kinds {
            let n = conn.execute(
                "DELETE FROM _skipped WHERE kind = ?1",
                rusqlite::params![k.as_str()],
            )?;
            total += n as u64;
        }
        Ok(total)
    }
```

Append these free functions to the bottom of `db.rs` (outside `impl Db`):

```rust
fn csv_escape(s: &str) -> String {
    if s.contains([',', '"', '\n']) {
        format!("\"{}\"", s.replace('"', "\"\""))
    } else {
        s.to_string()
    }
}

fn sql_escape(s: &str) -> String {
    s.replace('\'', "''")
}
```

#### 14.0.3 — `scraper.rs` additions

Append to the bottom of `turtle-scraper/src-tauri/src/scraper.rs`:

```rust
/// Run a targeted scrape for only the given (kind, entry) pairs.
/// Used for "scrape missing / gaps" mode.
pub async fn run_targeted(
    app: AppHandle,
    engine: Arc<Engine>,
    db: Arc<Db>,
    gaps: Vec<crate::types::GapEntry>,
    cfg: ScrapeConfig,
) -> Result<()> {
    if engine.running.swap(true, std::sync::atomic::Ordering::SeqCst) {
        return Err(anyhow!("already running"));
    }
    engine.cancel.store(false, Ordering::Relaxed);
    engine.paused.store(false, Ordering::Relaxed);
    engine.challenge.store(false, Ordering::Relaxed);

    let concurrency = cfg.concurrency.min(HARD_CONCURRENCY_CAP).max(1);
    let rate_per_sec = cfg.rate_per_sec.min(HARD_RATE_CAP).max(1);
    let limiter = build_limiter(rate_per_sec);
    let sem = Arc::new(Semaphore::new(concurrency as usize));
    let client = build_client()?;

    let total_queued = gaps.len() as u64;
    let done = Arc::new(AtomicU64::new(0));
    let found = Arc::new(AtomicU64::new(0));
    let skipped = Arc::new(AtomicU64::new(0));
    let errors = Arc::new(AtomicU64::new(0));
    let started = Instant::now();
    let mut futs = FuturesUnordered::new();

    for gap in gaps {
        if engine.cancel.load(Ordering::Relaxed) {
            break;
        }
        let kind_val = match gap.kind.as_str() {
            "item" => Kind::Item,
            "spell" => Kind::Spell,
            "quest" => Kind::Quest,
            "npc" => Kind::Npc,
            _ => Kind::Object,
        };
        let id = gap.entry;
        let permit = sem.clone().acquire_owned().await.unwrap();
        let (client, limiter, db, app, engine) = (
            client.clone(), limiter.clone(), db.clone(), app.clone(), engine.clone(),
        );
        let (done, found, skipped, errors) = (
            done.clone(), found.clone(), skipped.clone(), errors.clone(),
        );
        futs.push(tokio::spawn(async move {
            let _permit = permit;
            let url = kind_val.url(id);
            match fetch_with_retry(&client, &limiter, &url, &engine).await {
                Ok((200, body, _)) => match parse(kind_val, id, &body) {
                    Ok(ParseOutcome::Ok(rec)) => {
                        if db.upsert(&rec).is_ok() { found.fetch_add(1, Ordering::Relaxed); }
                        else { errors.fetch_add(1, Ordering::Relaxed); }
                    }
                    Ok(ParseOutcome::NotFound) => {
                        db.mark_skipped(kind_val, id, "not-found").ok();
                        skipped.fetch_add(1, Ordering::Relaxed);
                    }
                    Err(_) => { errors.fetch_add(1, Ordering::Relaxed); }
                },
                Ok((status, _, _)) => {
                    db.mark_skipped(kind_val, id, &format!("http-{status}")).ok();
                    skipped.fetch_add(1, Ordering::Relaxed);
                }
                Err(_) => { errors.fetch_add(1, Ordering::Relaxed); }
            }
            let d = done.fetch_add(1, Ordering::Relaxed) + 1;
            let rps = d as f64 / started.elapsed().as_secs_f64().max(0.001);
            let _ = app.emit("progress", crate::types::Progress {
                kind: kind_val.as_str().into(),
                queued: total_queued,
                done: d,
                found: found.load(Ordering::Relaxed),
                skipped: skipped.load(Ordering::Relaxed),
                errors: errors.load(Ordering::Relaxed),
                current_id: id,
                req_per_sec: rps,
                state: if engine.cancel.load(Ordering::Relaxed) { "cancelled" }
                       else if engine.challenge.load(Ordering::Relaxed) { "challenge" }
                       else if engine.paused.load(Ordering::Relaxed) { "paused" }
                       else { "running" },
            });
        }));
        while futs.len() >= concurrency as usize * 4 {
            if futs.next().await.is_none() { break; }
        }
    }
    while futs.next().await.is_some() {}
    let final_state = if engine.cancel.load(Ordering::Relaxed) { "cancelled" } else { "done" };
    let _ = app.emit("progress", crate::types::Progress {
        kind: "all".into(),
        queued: total_queued,
        done: done.load(Ordering::Relaxed),
        found: found.load(Ordering::Relaxed),
        skipped: skipped.load(Ordering::Relaxed),
        errors: errors.load(Ordering::Relaxed),
        current_id: 0,
        req_per_sec: 0.0,
        state: final_state,
    });
    engine.running.store(false, std::sync::atomic::Ordering::SeqCst);
    Ok(())
}
```

#### 14.0.4 — Replace `lib.rs` entirely

Replace the full contents of `turtle-scraper/src-tauri/src/lib.rs` with:

```rust
//! Tauri entry. Exposes all commands the frontend calls via invoke().

mod db;
mod parser;
mod rate;
mod scraper;
mod types;

use std::path::PathBuf;
use std::sync::Arc;
use tauri::{Manager, State};
use tracing_subscriber::EnvFilter;

use crate::db::Db;
use crate::scraper::Engine;
use crate::types::{GapEntry, Record, RichStats, ScrapeConfig, ScrapeProfile};

pub struct AppState {
    pub db: Arc<Db>,
    pub engine: Arc<Engine>,
}

fn db_path(app: &tauri::AppHandle) -> PathBuf {
    let dir = app
        .path()
        .app_data_dir()
        .unwrap_or_else(|_| PathBuf::from("."));
    std::fs::create_dir_all(&dir).ok();
    dir.join("turtle.db")
}

// ── Scrape control ────────────────────────────────────────────────────────────

#[tauri::command]
async fn start_scrape(
    app: tauri::AppHandle,
    state: State<'_, AppState>,
    cfg: ScrapeConfig,
) -> Result<(), String> {
    let engine = state.engine.clone();
    let db = state.db.clone();
    tokio::spawn(async move {
        if let Err(e) = scraper::run(app, engine, db, cfg).await {
            tracing::error!("scrape failed: {e}");
        }
    });
    Ok(())
}

#[tauri::command]
async fn start_targeted_scrape(
    app: tauri::AppHandle,
    state: State<'_, AppState>,
    gaps: Vec<GapEntry>,
    cfg: ScrapeConfig,
) -> Result<(), String> {
    let engine = state.engine.clone();
    let db = state.db.clone();
    tokio::spawn(async move {
        if let Err(e) = scraper::run_targeted(app, engine, db, gaps, cfg).await {
            tracing::error!("targeted scrape failed: {e}");
        }
    });
    Ok(())
}

#[tauri::command]
fn pause_scrape(state: State<'_, AppState>) {
    state.engine.paused.store(true, std::sync::atomic::Ordering::Relaxed);
}

#[tauri::command]
fn resume_scrape(state: State<'_, AppState>) {
    state.engine.paused.store(false, std::sync::atomic::Ordering::Relaxed);
    state.engine.challenge.store(false, std::sync::atomic::Ordering::Relaxed);
}

#[tauri::command]
fn cancel_scrape(state: State<'_, AppState>) {
    state.engine.cancel.store(true, std::sync::atomic::Ordering::Relaxed);
}

#[tauri::command]
fn engine_state(state: State<'_, AppState>) -> serde_json::Value {
    use std::sync::atomic::Ordering;
    serde_json::json!({
        "running": state.engine.running.load(Ordering::Relaxed),
        "paused":  state.engine.paused.load(Ordering::Relaxed),
        "challenge": state.engine.challenge.load(Ordering::Relaxed),
    })
}

// ── Database queries ──────────────────────────────────────────────────────────

#[tauri::command]
fn db_stats(state: State<'_, AppState>) -> Result<serde_json::Value, String> {
    let total = state.db.count(None).map_err(|e| e.to_string())?;
    let mut by_kind = serde_json::Map::new();
    for k in types::Kind::all() {
        let n = state.db.count(Some(k)).map_err(|e| e.to_string())?;
        by_kind.insert(k.as_str().to_string(), serde_json::json!(n));
    }
    Ok(serde_json::json!({
        "total": total,
        "by_kind": by_kind,
        "path": state.db.path().to_string_lossy(),
    }))
}

#[tauri::command]
fn rich_stats(state: State<'_, AppState>) -> Result<RichStats, String> {
    state.db.rich_stats().map_err(|e| e.to_string())
}

#[tauri::command]
fn search(state: State<'_, AppState>, q: String, limit: u32) -> Result<Vec<Record>, String> {
    state.db.search(&q, limit.min(500)).map_err(|e| e.to_string())
}

#[tauri::command]
fn get_record(
    state: State<'_, AppState>,
    kind: String,
    entry: u32,
) -> Result<Option<Record>, String> {
    let k = parse_kind(&kind)?;
    state.db.get_record(k, entry).map_err(|e| e.to_string())
}

#[tauri::command]
fn gap_ids(
    state: State<'_, AppState>,
    kinds: Vec<String>,
    start_id: u32,
    end_id: u32,
) -> Result<Vec<GapEntry>, String> {
    let ks: Result<Vec<types::Kind>, String> = kinds.iter().map(|s| parse_kind(s)).collect();
    state.db.gap_ids(&ks?, start_id, end_id).map_err(|e| e.to_string())
}

#[tauri::command]
fn clear_skipped(state: State<'_, AppState>, kinds: Vec<String>) -> Result<u64, String> {
    let ks: Result<Vec<types::Kind>, String> = kinds.iter().map(|s| parse_kind(s)).collect();
    state.db.clear_skipped(&ks?).map_err(|e| e.to_string())
}

// ── Export ────────────────────────────────────────────────────────────────────

#[tauri::command]
fn export_json(state: State<'_, AppState>, path: String) -> Result<u64, String> {
    state.db.export_json(std::path::Path::new(&path)).map_err(|e| e.to_string())
}

#[tauri::command]
fn export_csv(state: State<'_, AppState>, path: String, kind_filter: Option<String>) -> Result<u64, String> {
    let kf = kind_filter.as_deref().map(|s| parse_kind(s)).transpose()?;
    let bytes = state.db.export_csv(kf).map_err(|e| e.to_string())?;
    let n = bytes.iter().filter(|&&b| b == b'\n').count().saturating_sub(1) as u64;
    std::fs::write(&path, bytes).map_err(|e| e.to_string())?;
    Ok(n)
}

#[tauri::command]
fn export_sql(state: State<'_, AppState>, path: String, kind_filter: Option<String>) -> Result<u64, String> {
    let kf = kind_filter.as_deref().map(|s| parse_kind(s)).transpose()?;
    let bytes = state.db.export_sql(kf).map_err(|e| e.to_string())?;
    let n = bytes.windows(b"INSERT".len()).filter(|w| *w == b"INSERT").count() as u64;
    std::fs::write(&path, bytes).map_err(|e| e.to_string())?;
    Ok(n)
}

// ── Import ────────────────────────────────────────────────────────────────────

#[tauri::command]
fn import_json(state: State<'_, AppState>, path: String) -> Result<serde_json::Value, String> {
    let bytes = std::fs::read(&path).map_err(|e| e.to_string())?;
    let (inserted, updated, skipped) = state.db.import_json(&bytes).map_err(|e| e.to_string())?;
    Ok(serde_json::json!({ "inserted": inserted, "updated": updated, "skipped": skipped }))
}

// ── Profiles ──────────────────────────────────────────────────────────────────

#[tauri::command]
fn save_profile(state: State<'_, AppState>, profile: ScrapeProfile) -> Result<(), String> {
    state.db.save_profile(&profile).map_err(|e| e.to_string())
}

#[tauri::command]
fn load_profiles(state: State<'_, AppState>) -> Result<Vec<ScrapeProfile>, String> {
    state.db.load_profiles().map_err(|e| e.to_string())
}

#[tauri::command]
fn delete_profile(state: State<'_, AppState>, name: String) -> Result<(), String> {
    state.db.delete_profile(&name).map_err(|e| e.to_string())
}

// ── Helpers ───────────────────────────────────────────────────────────────────

fn parse_kind(s: &str) -> Result<types::Kind, String> {
    match s {
        "item" => Ok(types::Kind::Item),
        "spell" => Ok(types::Kind::Spell),
        "quest" => Ok(types::Kind::Quest),
        "npc" => Ok(types::Kind::Npc),
        "object" => Ok(types::Kind::Object),
        other => Err(format!("unknown kind: {other}")),
    }
}

// ── Tauri init ────────────────────────────────────────────────────────────────

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tracing_subscriber::fmt()
        .with_env_filter(
            EnvFilter::try_from_default_env().unwrap_or_else(|_| EnvFilter::new("info")),
        )
        .init();

    tauri::Builder::default()
        .plugin(tauri_plugin_shell::init())
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_fs::init())
        .setup(|app| {
            let path = db_path(&app.handle());
            let db = Arc::new(Db::open(&path).expect("open sqlite"));
            app.manage(AppState {
                db,
                engine: Engine::new(),
            });
            Ok(())
        })
        .invoke_handler(tauri::generate_handler![
            start_scrape,
            start_targeted_scrape,
            pause_scrape,
            resume_scrape,
            cancel_scrape,
            engine_state,
            db_stats,
            rich_stats,
            search,
            get_record,
            gap_ids,
            clear_skipped,
            export_json,
            export_csv,
            export_sql,
            import_json,
            save_profile,
            load_profiles,
            delete_profile,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

✅ Verify: `cargo check --manifest-path src-tauri/Cargo.toml` — zero errors.

---

### 14.1 — Replace `turtle-scraper/index.html`

```html
<!doctype html>
<html lang="en" data-theme="dark">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Turtle Scraper</title>
    <link rel="stylesheet" href="/src/styles.css" />
  </head>
  <body>
    <!-- ── Top bar ─────────────────────────────────────── -->
    <header id="topbar">
      <span class="logo">🐢 Turtle Scraper</span>
      <nav id="tabs" role="tablist">
        <button role="tab" aria-selected="true"  data-tab="scrape"    class="tab active">Scrape</button>
        <button role="tab" aria-selected="false" data-tab="browse"    class="tab">Browse</button>
        <button role="tab" aria-selected="false" data-tab="export"    class="tab">Export</button>
        <button role="tab" aria-selected="false" data-tab="import"    class="tab">Import</button>
        <button role="tab" aria-selected="false" data-tab="profiles"  class="tab">Profiles</button>
        <button role="tab" aria-selected="false" data-tab="stats"     class="tab">Stats</button>
      </nav>
      <div id="topbar-right">
        <span id="dbpill" class="pill" title="Records in database">0</span>
        <button id="theme-btn" title="Toggle theme" aria-label="Toggle theme">☀</button>
      </div>
    </header>

    <!-- ── Tab: Scrape ────────────────────────────────── -->
    <section id="tab-scrape" class="tabpanel active" role="tabpanel">
      <div class="two-col">
        <div class="col-left">
          <div class="card">
            <h2>Configuration</h2>
            <form id="cfg-form" class="cfg-grid">
              <label class="full-row">Kinds to scrape
                <div class="kind-checks" id="kind-checks">
                  <label><input type="checkbox" name="kind" value="item"   checked /> Item</label>
                  <label><input type="checkbox" name="kind" value="spell"          /> Spell</label>
                  <label><input type="checkbox" name="kind" value="quest"          /> Quest</label>
                  <label><input type="checkbox" name="kind" value="npc"            /> NPC</label>
                  <label><input type="checkbox" name="kind" value="object"         /> Object</label>
                </div>
              </label>
              <label>Start ID <input id="start" type="number" value="1"     min="1" /></label>
              <label>End ID   <input id="end"   type="number" value="90000" min="1" /></label>
              <label>Concurrency
                <input id="conc" type="range" min="1" max="16" value="8" />
                <output id="conc-out">8</output>
              </label>
              <label>Rate / sec
                <input id="rate" type="range" min="1" max="24" value="12" />
                <output id="rate-out">12</output>
              </label>
              <label class="full-row checkbox-row">
                <input id="use-sitemap" type="checkbox" checked />
                Use sitemap for known IDs first (faster)
              </label>
              <label class="full-row checkbox-row">
                <input id="skip-known" type="checkbox" checked />
                Skip IDs already in database
              </label>
              <label class="full-row checkbox-row">
                <input id="rescrape-skipped" type="checkbox" />
                Re-attempt previously skipped (not-found) IDs
              </label>
            </form>
            <div class="btn-row">
              <button id="start-btn"  class="btn-primary">▶ Start</button>
              <button id="pause-btn"  class="btn-sec" disabled>⏸ Pause</button>
              <button id="resume-btn" class="btn-sec" disabled>▶ Resume</button>
              <button id="cancel-btn" class="btn-danger" disabled>✕ Cancel</button>
            </div>
            <div class="btn-row">
              <button id="gap-btn"    class="btn-sec">🔍 Scrape Missing IDs</button>
              <button id="save-profile-btn" class="btn-sec">💾 Save Profile</button>
            </div>
          </div>
        </div>

        <div class="col-right">
          <div class="card flex-card">
            <h2>Live Progress</h2>
            <div class="progress-wrap">
              <div class="progress-bar-bg">
                <div id="bar" class="progress-bar"></div>
              </div>
              <span id="pct-label">0%</span>
            </div>
            <div id="stat-grid" class="stat-grid">
              <div class="stat"><span class="stat-val" id="s-done">0</span><span class="stat-lbl">Done</span></div>
              <div class="stat"><span class="stat-val" id="s-found">0</span><span class="stat-lbl">Found</span></div>
              <div class="stat"><span class="stat-val" id="s-skip">0</span><span class="stat-lbl">Skipped</span></div>
              <div class="stat"><span class="stat-val" id="s-err">0</span><span class="stat-lbl">Errors</span></div>
              <div class="stat"><span class="stat-val" id="s-rps">0</span><span class="stat-lbl">Req/s</span></div>
              <div class="stat"><span class="stat-val" id="s-eta">—</span><span class="stat-lbl">ETA</span></div>
              <div class="stat"><span class="stat-val" id="s-elapsed">0s</span><span class="stat-lbl">Elapsed</span></div>
              <div class="stat"><span class="stat-val" id="s-id">—</span><span class="stat-lbl">Current ID</span></div>
            </div>
            <pre id="status" class="status-line">idle</pre>
            <canvas id="rps-chart" height="60" aria-label="Requests per second graph" role="img"></canvas>
          </div>

          <!-- Cloudflare challenge banner -->
          <div id="challenge" class="card warn-card hidden">
            <h2>⚠ Cloudflare Challenge</h2>
            <p>Open the site in your browser, complete the check, then click Resume.</p>
            <div class="btn-row">
              <button id="open-site" class="btn-sec">Open database.turtlecraft.gg</button>
              <button id="cf-resume" class="btn-primary">Resume after solve</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Session log -->
      <div class="card log-card">
        <div style="display:flex;justify-content:space-between;align-items:baseline">
          <h2>Session Log</h2>
          <button id="clear-log-btn" class="btn-ghost">Clear</button>
        </div>
        <div id="log" class="log"></div>
      </div>
    </section>

    <!-- ── Tab: Browse ────────────────────────────────── -->
    <section id="tab-browse" class="tabpanel" role="tabpanel" hidden>
      <div class="card">
        <h2>Search &amp; Browse</h2>
        <div class="search-row">
          <input id="q" placeholder="search by name…" autocomplete="off" />
          <select id="kind-filter">
            <option value="">All kinds</option>
            <option value="item">Item</option>
            <option value="spell">Spell</option>
            <option value="quest">Quest</option>
            <option value="npc">NPC</option>
            <option value="object">Object</option>
          </select>
          <input id="q-entry" type="number" placeholder="exact ID…" min="1" />
          <button id="search-btn" class="btn-primary">Search</button>
        </div>
        <div id="results" class="results-grid"></div>
      </div>

      <!-- Detail panel -->
      <div id="detail-panel" class="card detail-card hidden">
        <div style="display:flex;justify-content:space-between;align-items:baseline">
          <h2 id="detail-title">—</h2>
          <button id="detail-close" class="btn-ghost" aria-label="Close">✕</button>
        </div>
        <div id="detail-meta" class="detail-meta"></div>
        <div id="detail-fields" class="detail-fields"></div>
        <details>
          <summary class="muted">Raw tooltip HTML</summary>
          <div id="detail-tooltip" class="tooltip-preview"></div>
        </details>
      </div>
    </section>

    <!-- ── Tab: Export ────────────────────────────────── -->
    <section id="tab-export" class="tabpanel" role="tabpanel" hidden>
      <div class="card">
        <h2>Export Database</h2>
        <p class="muted">All formats include full field data. Exports run without locking the scraper.</p>
        <div class="export-grid">
          <div class="export-item">
            <strong>JSON</strong>
            <p class="muted">Array of all records with every field. Re-importable.</p>
            <div class="btn-row">
              <button class="btn-primary" id="exp-json-all">Export all</button>
              <select id="exp-json-kind"><option value="">All kinds</option><option value="item">Item</option><option value="spell">Spell</option><option value="quest">Quest</option><option value="npc">NPC</option><option value="object">Object</option></select>
            </div>
          </div>
          <div class="export-item">
            <strong>CSV</strong>
            <p class="muted">Spreadsheet-compatible. kind, entry, name, category, quality, fetched_at.</p>
            <div class="btn-row">
              <button class="btn-sec" id="exp-csv-all">Export all</button>
              <select id="exp-csv-kind"><option value="">All kinds</option><option value="item">Item</option><option value="spell">Spell</option><option value="quest">Quest</option><option value="npc">NPC</option><option value="object">Object</option></select>
            </div>
          </div>
          <div class="export-item">
            <strong>SQL</strong>
            <p class="muted">INSERT OR REPLACE statements for any SQLite-compatible database.</p>
            <div class="btn-row">
              <button class="btn-sec" id="exp-sql-all">Export all</button>
              <select id="exp-sql-kind"><option value="">All kinds</option><option value="item">Item</option><option value="spell">Spell</option><option value="quest">Quest</option><option value="npc">NPC</option><option value="object">Object</option></select>
            </div>
          </div>
          <div class="export-item">
            <strong>SQLite .db file</strong>
            <p class="muted">Copy the live database file to a chosen location.</p>
            <div class="btn-row">
              <button class="btn-sec" id="exp-db">Copy .db file</button>
            </div>
          </div>
        </div>
        <pre id="export-status" class="status-line"></pre>
      </div>
    </section>

    <!-- ── Tab: Import ────────────────────────────────── -->
    <section id="tab-import" class="tabpanel" role="tabpanel" hidden>
      <div class="card">
        <h2>Import JSON Archive</h2>
        <p class="muted">Merges a previously exported JSON file into the current database.
          Existing records are updated only if the import's <code>fetched_at</code> is newer.</p>
        <div class="btn-row">
          <button id="import-btn" class="btn-primary">Choose JSON file…</button>
        </div>
        <pre id="import-status" class="status-line"></pre>
      </div>
      <div class="card">
        <h2>Verify &amp; Enrich</h2>
        <p class="muted">After importing, find IDs that are in the import file but have stale or missing data in the DB, then scrape only those.</p>
        <div class="cfg-grid">
          <label>Start ID <input id="enrich-start" type="number" value="1" min="1" /></label>
          <label>End ID   <input id="enrich-end" type="number" value="90000" min="1" /></label>
        </div>
        <div class="btn-row">
          <button id="find-gaps-btn" class="btn-sec">Find gaps / missing</button>
          <button id="scrape-gaps-btn" class="btn-primary" disabled>Scrape missing now</button>
        </div>
        <pre id="gaps-status" class="status-line"></pre>
        <div id="gaps-list" class="gaps-list"></div>
      </div>
    </section>

    <!-- ── Tab: Profiles ─────────────────────────────── -->
    <section id="tab-profiles" class="tabpanel" role="tabpanel" hidden>
      <div class="card">
        <h2>Saved Profiles</h2>
        <p class="muted">Save the current scrape configuration under a name to recall it instantly.</p>
        <div id="profiles-list" class="profiles-list"></div>
        <p class="muted" id="no-profiles">No profiles saved yet. Configure a scrape and click "Save Profile" in the Scrape tab.</p>
      </div>
    </section>

    <!-- ── Tab: Stats ─────────────────────────────────── -->
    <section id="tab-stats" class="tabpanel" role="tabpanel" hidden>
      <div class="card">
        <h2>Database Statistics</h2>
        <div id="stats-body" class="stats-body">
          <div class="stat-block">
            <div class="big-num" id="stats-total">—</div>
            <div class="muted">Total records</div>
          </div>
          <div id="kind-bars" class="kind-bars"></div>
        </div>
        <table id="stats-table" class="stats-table">
          <thead><tr><th>Kind</th><th>Records</th><th>Skipped</th></tr></thead>
          <tbody id="stats-rows"></tbody>
        </table>
        <div class="stats-meta" id="stats-meta"></div>
        <div class="btn-row" style="margin-top:1rem">
          <button id="refresh-stats-btn" class="btn-sec">↻ Refresh</button>
          <button id="clear-skipped-btn" class="btn-ghost">Clear all skipped entries</button>
        </div>
      </div>
    </section>

    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

---

### 14.2 — Replace `turtle-scraper/src/styles.css`

```css
/* ── Design tokens ─────────────────────────────────────── */
:root {
  --bg:        #0c0e14;
  --bg2:       #11141c;
  --panel:     #161b24;
  --border:    #222a38;
  --border2:   #2d3748;
  --text:      #e2e8f4;
  --muted:     #7a8799;
  --accent:    #4ade80;
  --accent2:   #22d3ee;
  --warn:      #fbbf24;
  --err:       #f87171;
  --danger:    #ef4444;
  --radius:    6px;
  --shadow:    0 2px 12px rgba(0,0,0,.45);
  font-family: ui-sans-serif, "Segoe UI", system-ui, sans-serif;
  font-size:   14px;
  color-scheme: dark;
}
[data-theme="light"] {
  --bg:    #f0f2f7;
  --bg2:   #ffffff;
  --panel: #ffffff;
  --border:#cbd5e1;
  --border2:#94a3b8;
  --text:  #1a202c;
  --muted: #64748b;
  color-scheme: light;
}

/* ── Reset ─────────────────────────────────────────────── */
*, *::before, *::after { box-sizing: border-box; margin: 0; }
html, body { height: 100%; background: var(--bg); color: var(--text); overflow-x: hidden; }
a { color: var(--accent2); }
button, input, select, textarea { font: inherit; }

/* ── Top bar ───────────────────────────────────────────── */
#topbar {
  position: sticky; top: 0; z-index: 100;
  display: flex; align-items: center; gap: 1rem;
  padding: .5rem 1.25rem;
  background: var(--bg2); border-bottom: 1px solid var(--border);
}
.logo { font-weight: 700; font-size: 1rem; white-space: nowrap; }
#tabs { display: flex; gap: 2px; }
.tab {
  background: none; border: none; border-radius: var(--radius);
  padding: .35rem .75rem; color: var(--muted); cursor: pointer;
  transition: background .15s, color .15s;
}
.tab:hover { background: var(--border); color: var(--text); }
.tab.active { background: var(--border2); color: var(--text); font-weight: 600; }
#topbar-right { margin-left: auto; display: flex; align-items: center; gap: .5rem; }
.pill {
  background: var(--panel); border: 1px solid var(--border2);
  border-radius: 99px; padding: .1rem .65rem; font-size: .8rem; color: var(--accent);
}
#theme-btn { background: none; border: none; cursor: pointer; font-size: 1.1rem; padding: .2rem; }

/* ── Layout ────────────────────────────────────────────── */
.tabpanel { padding: 1rem 1.25rem; max-width: 1400px; margin: 0 auto; }
.two-col { display: grid; grid-template-columns: 380px 1fr; gap: 1rem; align-items: start; }
@media (max-width: 900px) { .two-col { grid-template-columns: 1fr; } }
.col-left, .col-right { display: flex; flex-direction: column; gap: 1rem; }
.flex-card { display: flex; flex-direction: column; gap: .75rem; }

/* ── Card ──────────────────────────────────────────────── */
.card {
  background: var(--panel); border: 1px solid var(--border);
  border-radius: calc(var(--radius) * 1.5); padding: 1rem 1.25rem;
  box-shadow: var(--shadow);
}
.card h2 { font-size: .8rem; font-weight: 700; text-transform: uppercase;
  letter-spacing: .07em; color: var(--muted); margin-bottom: .75rem; }
.warn-card { border-color: var(--warn); background: color-mix(in srgb, var(--warn) 8%, var(--panel)); }
.log-card { margin-top: 1rem; }

/* ── Config form ───────────────────────────────────────── */
.cfg-grid {
  display: grid; grid-template-columns: 1fr 1fr; gap: .65rem .9rem;
}
.cfg-grid label { display: flex; flex-direction: column; gap: .25rem; font-size: .82rem; color: var(--muted); }
.cfg-grid label input, .cfg-grid label select { color: var(--text); }
.full-row { grid-column: 1 / -1; }
.checkbox-row { flex-direction: row !important; align-items: center; gap: .5rem; color: var(--text) !important; }
input[type="range"] { accent-color: var(--accent); width: 100%; }
output { font-size: .8rem; color: var(--accent); text-align: right; }
.kind-checks { display: flex; flex-wrap: wrap; gap: .5rem .9rem; margin-top: .3rem; }
.kind-checks label { flex-direction: row; align-items: center; gap: .3rem; color: var(--text); }

/* ── Inputs / selects ──────────────────────────────────── */
input[type="text"], input[type="number"], input[type="search"], select, textarea {
  background: var(--bg); color: var(--text);
  border: 1px solid var(--border2); border-radius: var(--radius);
  padding: .4rem .65rem; width: 100%;
}
input:focus, select:focus { outline: 2px solid var(--accent); outline-offset: -1px; border-color: transparent; }

/* ── Buttons ───────────────────────────────────────────── */
.btn-row { display: flex; gap: .5rem; flex-wrap: wrap; margin-top: .5rem; }
button { cursor: pointer; border-radius: var(--radius); border: 1px solid transparent;
  padding: .4rem .85rem; font-size: .82rem; transition: filter .12s, opacity .12s; }
button:disabled { opacity: .38; cursor: not-allowed; }
.btn-primary { background: var(--accent); color: #0a1a10; border-color: var(--accent); font-weight: 700; }
.btn-primary:hover:not(:disabled) { filter: brightness(1.1); }
.btn-sec { background: var(--border2); color: var(--text); border-color: var(--border2); }
.btn-sec:hover:not(:disabled) { filter: brightness(1.15); }
.btn-danger { background: var(--danger); color: #fff; border-color: var(--danger); }
.btn-danger:hover:not(:disabled) { filter: brightness(1.1); }
.btn-ghost { background: none; border-color: var(--border2); color: var(--muted); }
.btn-ghost:hover:not(:disabled) { color: var(--text); border-color: var(--border); }

/* ── Progress ──────────────────────────────────────────── */
.progress-wrap { display: flex; align-items: center; gap: .75rem; }
.progress-bar-bg { flex: 1; height: 8px; background: var(--border); border-radius: 4px; overflow: hidden; }
.progress-bar { height: 100%; width: 0%; background: var(--accent); transition: width .25s linear; border-radius: 4px; }
#pct-label { font-size: .8rem; color: var(--accent); min-width: 3rem; text-align: right; }
.stat-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: .5rem; }
.stat { background: var(--bg); border: 1px solid var(--border); border-radius: var(--radius);
  padding: .5rem .65rem; display: flex; flex-direction: column; gap: .1rem; }
.stat-val { font-size: 1.1rem; font-weight: 700; color: var(--text); font-variant-numeric: tabular-nums; }
.stat-lbl { font-size: .72rem; color: var(--muted); }
.status-line { font-family: ui-monospace, "Cascadia Mono", monospace; font-size: .78rem;
  color: var(--muted); white-space: pre-wrap; min-height: 1.4em; margin-top: .25rem; }
#rps-chart { width: 100%; border-radius: var(--radius); background: var(--bg); margin-top: .25rem; }

/* ── Log ───────────────────────────────────────────────── */
.log { max-height: 200px; overflow-y: auto; font-family: ui-monospace, "Cascadia Mono", monospace;
  font-size: .77rem; display: flex; flex-direction: column; gap: 2px; }
.log-entry { padding: 2px 4px; border-radius: 3px; }
.log-info  { color: var(--muted); }
.log-warn  { color: var(--warn); }
.log-error { color: var(--err); }

/* ── Browse ────────────────────────────────────────────── */
.search-row { display: flex; gap: .5rem; flex-wrap: wrap; margin-bottom: .75rem; }
.search-row input, .search-row select { flex: 1; min-width: 120px; }
.results-grid { display: flex; flex-direction: column; gap: 3px; max-height: 420px; overflow-y: auto; }
.result-row {
  display: grid; grid-template-columns: 70px 80px 1fr 130px 70px;
  gap: .5rem; padding: .3rem .5rem;
  border: 1px solid var(--border); border-radius: var(--radius);
  font-size: .85rem; cursor: pointer; transition: background .1s;
}
.result-row:hover { background: var(--border); }
.q0 { color: #9d9d9d; } .q1 { color: #fff; } .q2 { color: #1eff00; }
.q3 { color: #0070dd; } .q4 { color: #a335ee; } .q5 { color: #ff8000; }

/* ── Detail panel ──────────────────────────────────────── */
.detail-card { margin-top: 1rem; }
.detail-meta { display: flex; gap: 1rem; font-size: .82rem; color: var(--muted); flex-wrap: wrap; margin-bottom: .75rem; }
.detail-meta span strong { color: var(--text); }
.detail-fields { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: .5rem; margin-bottom: .75rem; }
.field-chip { background: var(--bg); border: 1px solid var(--border); border-radius: var(--radius);
  padding: .35rem .6rem; font-size: .8rem; }
.field-chip .fk { color: var(--muted); font-size: .72rem; margin-bottom: .15rem; }
.field-chip .fv { color: var(--text); word-break: break-word; }
.tooltip-preview { padding: .5rem; overflow-x: auto; }
/* Mirror site quality colours for tooltips */
.tooltip-preview table { border-collapse: collapse; font-size: .82rem; }
.tooltip-preview td, .tooltip-preview th { padding: .2rem .5rem; border: 1px solid var(--border2); }

/* ── Export ────────────────────────────────────────────── */
.export-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 1rem; margin-bottom: .75rem; }
.export-item { background: var(--bg); border: 1px solid var(--border);
  border-radius: var(--radius); padding: .75rem; display: flex; flex-direction: column; gap: .4rem; }

/* ── Import ────────────────────────────────────────────── */
.gaps-list { display: flex; flex-wrap: wrap; gap: .3rem; max-height: 120px; overflow-y: auto; margin-top: .5rem; }
.gap-chip { background: var(--bg); border: 1px solid var(--border2);
  border-radius: 3px; padding: .15rem .4rem; font-size: .75rem; color: var(--muted); }

/* ── Profiles ──────────────────────────────────────────── */
.profiles-list { display: flex; flex-direction: column; gap: .5rem; }
.profile-row { display: flex; align-items: center; gap: .75rem; padding: .45rem .65rem;
  background: var(--bg); border: 1px solid var(--border); border-radius: var(--radius); }
.profile-row .pname { flex: 1; font-size: .85rem; }
.profile-row .pmeta { font-size: .75rem; color: var(--muted); }

/* ── Stats ─────────────────────────────────────────────── */
.stats-body { display: flex; gap: 2rem; align-items: flex-start; flex-wrap: wrap; margin-bottom: 1rem; }
.stat-block { text-align: center; }
.big-num { font-size: 2.5rem; font-weight: 800; color: var(--accent); line-height: 1; }
.kind-bars { display: flex; flex-direction: column; gap: .35rem; flex: 1; min-width: 200px; }
.kind-bar-row { display: grid; grid-template-columns: 55px 1fr 60px; align-items: center; gap: .5rem; font-size: .8rem; }
.kind-bar-track { height: 10px; background: var(--border); border-radius: 5px; overflow: hidden; }
.kind-bar-fill  { height: 100%; background: var(--accent2); border-radius: 5px; transition: width .4s; }
.stats-table { width: 100%; border-collapse: collapse; font-size: .85rem; margin-bottom: .75rem; }
.stats-table th { text-align: left; padding: .4rem .5rem; border-bottom: 1px solid var(--border2); color: var(--muted); font-weight: 600; }
.stats-table td { padding: .35rem .5rem; border-bottom: 1px solid var(--border); }
.stats-meta { font-size: .78rem; color: var(--muted); line-height: 1.7; }

/* ── Misc ──────────────────────────────────────────────── */
.muted { color: var(--muted); }
.hidden { display: none !important; }
details summary { cursor: pointer; color: var(--muted); font-size: .8rem; padding: .35rem 0; }
```

---

### 14.3 — Replace `turtle-scraper/src/main.ts`

```ts
import { invoke } from "@tauri-apps/api/core";
import { listen } from "@tauri-apps/api/event";
import { save, open as openDialog } from "@tauri-apps/plugin-dialog";
import { open as openShell } from "@tauri-apps/plugin-shell";
import { copyFile } from "@tauri-apps/plugin-fs";

// ── Types ──────────────────────────────────────────────────────────────────────

type Kind = "item" | "spell" | "quest" | "npc" | "object";
const ALL_KINDS: Kind[] = ["item", "spell", "quest", "npc", "object"];

interface Progress {
  kind: string;
  queued: number;
  done: number;
  found: number;
  skipped: number;
  errors: number;
  current_id: number;
  req_per_sec: number;
  state: "idle" | "running" | "paused" | "challenge" | "done" | "cancelled" | "error";
}

interface ScrapedRecord {
  kind: Kind;
  entry: number;
  name: string;
  category: string | null;
  quality: number | null;
  fields: Record<string, unknown>;
  tooltip_html: string;
  fetched_at: number;
}

interface RichStats {
  total: number;
  by_kind: Record<string, number>;
  skipped_total: number;
  skipped_by_kind: Record<string, number>;
  db_size_bytes: number;
  path: string;
  oldest_fetch: number | null;
  newest_fetch: number | null;
}

interface ScrapeConfig {
  kinds: Kind[];
  start_id: number;
  end_id: number;
  concurrency: number;
  rate_per_sec: number;
  use_sitemap: boolean;
}

interface ScrapeProfile {
  name: string;
  cfg: ScrapeConfig;
}

interface GapEntry {
  kind: string;
  entry: number;
}

// ── Utility ────────────────────────────────────────────────────────────────────

function $<T extends HTMLElement = HTMLElement>(id: string): T {
  return document.getElementById(id) as T;
}

function esc(s: string): string {
  return s.replace(/[&<>"']/g, (c) =>
    ({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]!)
  );
}

function fmtNum(n: number): string {
  return n.toLocaleString();
}

function fmtBytes(b: number): string {
  if (b < 1024) return `${b} B`;
  if (b < 1024 * 1024) return `${(b / 1024).toFixed(1)} KB`;
  return `${(b / 1024 / 1024).toFixed(2)} MB`;
}

function fmtEta(done: number, total: number, rps: number): string {
  if (rps < 0.01 || done >= total) return "—";
  const secs = (total - done) / rps;
  if (secs < 60) return `${Math.round(secs)}s`;
  if (secs < 3600) return `${Math.round(secs / 60)}m`;
  const h = Math.floor(secs / 3600);
  const m = Math.round((secs % 3600) / 60);
  return `${h}h ${m}m`;
}

function fmtElapsed(ms: number): string {
  const s = Math.floor(ms / 1000);
  if (s < 60) return `${s}s`;
  if (s < 3600) return `${Math.floor(s/60)}m ${s%60}s`;
  return `${Math.floor(s/3600)}h ${Math.floor((s%3600)/60)}m`;
}

function tsToDate(unix: number): string {
  return new Date(unix * 1000).toLocaleString();
}

function addLog(msg: string, level: "info"|"warn"|"error" = "info") {
  const log = $("log");
  const el = document.createElement("div");
  el.className = `log-entry log-${level}`;
  const now = new Date().toLocaleTimeString();
  el.textContent = `[${now}] ${msg}`;
  log.appendChild(el);
  log.scrollTop = log.scrollHeight;
}

// ── Tabs ───────────────────────────────────────────────────────────────────────

const tabBtns = Array.from(document.querySelectorAll<HTMLButtonElement>(".tab"));
const tabPanels = Array.from(document.querySelectorAll<HTMLElement>(".tabpanel"));

function activateTab(name: string) {
  tabBtns.forEach(b => {
    const active = b.dataset.tab === name;
    b.classList.toggle("active", active);
    b.setAttribute("aria-selected", String(active));
  });
  tabPanels.forEach(p => {
    const active = p.id === `tab-${name}`;
    p.hidden = !active;
    p.classList.toggle("active", active);
  });
}

tabBtns.forEach(b => b.addEventListener("click", () => activateTab(b.dataset.tab!)));

// ── Theme ──────────────────────────────────────────────────────────────────────

let darkTheme = true;
$("theme-btn").addEventListener("click", () => {
  darkTheme = !darkTheme;
  document.documentElement.setAttribute("data-theme", darkTheme ? "dark" : "light");
  $("theme-btn").textContent = darkTheme ? "☀" : "🌙";
});

// ── RPS chart ─────────────────────────────────────────────────────────────────

const rpsCanvas = $<HTMLCanvasElement>("rps-chart");
const rpsCtx = rpsCanvas.getContext("2d")!;
const rpsHistory: number[] = [];
const RPS_MAX_POINTS = 120;

function drawRpsChart() {
  const W = rpsCanvas.offsetWidth;
  const H = 60;
  rpsCanvas.width = W;
  rpsCanvas.height = H;
  rpsCtx.clearRect(0, 0, W, H);
  if (rpsHistory.length < 2) return;
  const max = Math.max(...rpsHistory, 1);
  rpsCtx.beginPath();
  rpsCtx.strokeStyle = getComputedStyle(document.documentElement).getPropertyValue("--accent").trim();
  rpsCtx.lineWidth = 1.5;
  rpsHistory.forEach((v, i) => {
    const x = (i / (RPS_MAX_POINTS - 1)) * W;
    const y = H - (v / max) * (H - 4) - 2;
    i === 0 ? rpsCtx.moveTo(x, y) : rpsCtx.lineTo(x, y);
  });
  rpsCtx.stroke();
  // fill
  rpsCtx.lineTo(W, H); rpsCtx.lineTo(0, H); rpsCtx.closePath();
  rpsCtx.fillStyle = "rgba(74,222,128,.12)";
  rpsCtx.fill();
}

// ── Scrape state ───────────────────────────────────────────────────────────────

let scrapeStartMs = 0;
let elapsedInterval: ReturnType<typeof setInterval> | null = null;

function setRunning(running: boolean) {
  ($("start-btn") as HTMLButtonElement).disabled = running;
  ($("pause-btn") as HTMLButtonElement).disabled = !running;
  ($("resume-btn") as HTMLButtonElement).disabled = !running;
  ($("cancel-btn") as HTMLButtonElement).disabled = !running;
  ($("gap-btn") as HTMLButtonElement).disabled = running;
  if (running) {
    scrapeStartMs = Date.now();
    elapsedInterval = setInterval(() => {
      $("s-elapsed").textContent = fmtElapsed(Date.now() - scrapeStartMs);
    }, 1000);
  } else {
    if (elapsedInterval) { clearInterval(elapsedInterval); elapsedInterval = null; }
  }
}

function readConfig(): ScrapeConfig {
  const kinds = Array.from(
    document.querySelectorAll<HTMLInputElement>('input[name="kind"]:checked')
  ).map(cb => cb.value as Kind);
  return {
    kinds: kinds.length ? kinds : ["item"],
    start_id: Number(($("start") as HTMLInputElement).value),
    end_id: Number(($("end") as HTMLInputElement).value),
    concurrency: Number(($("conc") as HTMLInputElement).value),
    rate_per_sec: Number(($("rate") as HTMLInputElement).value),
    use_sitemap: ($("use-sitemap") as HTMLInputElement).checked,
  };
}

// Slider output labels
$<HTMLInputElement>("conc").addEventListener("input", (e) => {
  $("conc-out").textContent = (e.target as HTMLInputElement).value;
});
$<HTMLInputElement>("rate").addEventListener("input", (e) => {
  $("rate-out").textContent = (e.target as HTMLInputElement).value;
});

// ── Progress event ─────────────────────────────────────────────────────────────

await listen<Progress>("progress", (ev) => {
  const p = ev.payload;
  const pct = p.queued > 0 ? Math.min((p.done / p.queued) * 100, 100) : 0;
  ($("bar") as HTMLDivElement).style.width = `${pct.toFixed(1)}%`;
  $("pct-label").textContent = `${pct.toFixed(1)}%`;
  $("s-done").textContent  = fmtNum(p.done);
  $("s-found").textContent = fmtNum(p.found);
  $("s-skip").textContent  = fmtNum(p.skipped);
  $("s-err").textContent   = fmtNum(p.errors);
  $("s-rps").textContent   = p.req_per_sec.toFixed(1);
  $("s-eta").textContent   = fmtEta(p.done, p.queued, p.req_per_sec);
  $("s-id").textContent    = String(p.current_id || "—");
  $("status").textContent  =
    `${p.state.toUpperCase()}  ${fmtNum(p.done)}/${fmtNum(p.queued)}` +
    `  found=${fmtNum(p.found)}  errs=${fmtNum(p.errors)}  rps=${p.req_per_sec.toFixed(2)}`;

  rpsHistory.push(p.req_per_sec);
  if (rpsHistory.length > RPS_MAX_POINTS) rpsHistory.shift();
  drawRpsChart();

  if (p.state === "challenge") {
    $("challenge").classList.remove("hidden");
    addLog("Cloudflare challenge detected — solve in browser then Resume", "warn");
  }
  if (p.state === "done" || p.state === "cancelled" || p.state === "error") {
    setRunning(false);
    $("challenge").classList.add("hidden");
    refreshDbPill();
    addLog(`Scrape ${p.state}. Found ${fmtNum(p.found)}, skipped ${fmtNum(p.skipped)}, errors ${fmtNum(p.errors)}.`,
      p.state === "error" ? "error" : "info");
  }
});

// ── Scrape controls ────────────────────────────────────────────────────────────

$("start-btn").addEventListener("click", async () => {
  const cfg = readConfig();
  if (!cfg.kinds.length) { addLog("Select at least one kind.", "warn"); return; }
  try {
    setRunning(true);
    addLog(`Starting scrape: ${cfg.kinds.join(", ")} IDs ${cfg.start_id}–${cfg.end_id}`);
    await invoke("start_scrape", { cfg });
  } catch (err) {
    setRunning(false);
    addLog(`Failed to start: ${String(err)}`, "error");
  }
});

$("pause-btn").addEventListener("click", () => {
  invoke("pause_scrape");
  addLog("Paused.");
});

$("resume-btn").addEventListener("click", () => {
  $("challenge").classList.add("hidden");
  invoke("resume_scrape");
  addLog("Resumed.");
});

$("cancel-btn").addEventListener("click", () => {
  invoke("cancel_scrape");
  addLog("Cancel requested…", "warn");
});

$("cf-resume").addEventListener("click", () => {
  $("challenge").classList.add("hidden");
  invoke("resume_scrape");
  addLog("Resumed after Cloudflare solve.");
});

$("open-site").addEventListener("click", () =>
  openShell("https://database.turtlecraft.gg/")
);

$("clear-log-btn").addEventListener("click", () => { $("log").innerHTML = ""; });

// ── Gap / missing scrape ────────────────────────────────────────────────────────

let pendingGaps: GapEntry[] = [];

$("gap-btn").addEventListener("click", async () => {
  const cfg = readConfig();
  addLog("Finding gaps / missing IDs…");
  try {
    const gaps = await invoke<GapEntry[]>("gap_ids", {
      kinds: cfg.kinds,
      startId: cfg.start_id,
      endId: cfg.end_id,
    });
    pendingGaps = gaps;
    $("gaps-status").textContent = `Found ${gaps.length} missing IDs.`;
    renderGapsList(gaps);
    ($("scrape-gaps-btn") as HTMLButtonElement).disabled = gaps.length === 0;
    activateTab("import");
    addLog(`Found ${gaps.length} missing IDs — switch to Import tab to scrape them.`);
  } catch (err) {
    addLog(`gap_ids error: ${String(err)}`, "error");
  }
});

// ── Save profile ───────────────────────────────────────────────────────────────

$("save-profile-btn").addEventListener("click", async () => {
  const name = prompt("Profile name:");
  if (!name?.trim()) return;
  const cfg = readConfig();
  try {
    await invoke("save_profile", { profile: { name: name.trim(), cfg } });
    addLog(`Profile "${name.trim()}" saved.`);
    refreshProfiles();
  } catch (err) {
    addLog(`Save profile error: ${String(err)}`, "error");
  }
});

// ── Browse / Search ────────────────────────────────────────────────────────────

async function runSearch() {
  const q = ($("q") as HTMLInputElement).value.trim();
  const kf = ($("kind-filter") as HTMLSelectElement).value;
  const entryRaw = ($("q-entry") as HTMLInputElement).value.trim();

  // Exact-ID lookup
  if (entryRaw && kf) {
    try {
      const rec = await invoke<ScrapedRecord | null>("get_record", {
        kind: kf, entry: Number(entryRaw),
      });
      renderResults(rec ? [rec] : []);
      return;
    } catch (_) { /* fall through to name search */ }
  }

  if (!q) return;
  try {
    const rows = await invoke<ScrapedRecord[]>("search", { q, limit: 200 });
    renderResults(kf ? rows.filter(r => r.kind === kf) : rows);
  } catch (err) {
    addLog(`Search error: ${String(err)}`, "error");
  }
}

$("search-btn").addEventListener("click", runSearch);
$("q").addEventListener("keydown", (e) => { if (e.key === "Enter") runSearch(); });

function renderResults(rows: ScrapedRecord[]) {
  const container = $("results");
  if (!rows.length) {
    container.innerHTML = `<div class="muted" style="padding:.5rem">No results.</div>`;
    return;
  }
  container.innerHTML = rows.map(r => `
    <div class="result-row" data-kind="${esc(r.kind)}" data-entry="${r.entry}" tabindex="0" role="button"
         aria-label="${esc(r.name)}">
      <span class="muted">${esc(r.kind)}</span>
      <span class="muted">#${r.entry}</span>
      <span class="q${r.quality ?? 1}">${esc(r.name)}</span>
      <span class="muted">${esc(r.category ?? "")}</span>
      <span class="muted">${r.fetched_at ? tsToDate(r.fetched_at).split(",")[0] : ""}</span>
    </div>`).join("");

  container.querySelectorAll<HTMLDivElement>(".result-row").forEach(el => {
    el.addEventListener("click", () => openDetail(el.dataset.kind!, Number(el.dataset.entry!)));
    el.addEventListener("keydown", e => { if (e.key === "Enter") openDetail(el.dataset.kind!, Number(el.dataset.entry!)); });
  });
}

async function openDetail(kind: string, entry: number) {
  try {
    const rec = await invoke<ScrapedRecord | null>("get_record", { kind, entry });
    if (!rec) return;
    $("detail-title").innerHTML = `<span class="q${rec.quality ?? 1}">${esc(rec.name)}</span>`;
    $("detail-meta").innerHTML = `
      <span><strong>Kind:</strong> ${esc(rec.kind)}</span>
      <span><strong>ID:</strong> ${rec.entry}</span>
      <span><strong>Category:</strong> ${esc(rec.category ?? "—")}</span>
      <span><strong>Quality:</strong> ${rec.quality ?? "—"}</span>
      <span><strong>Fetched:</strong> ${rec.fetched_at ? tsToDate(rec.fetched_at) : "—"}</span>
    `;
    const fields = rec.fields as Record<string, unknown>;
    $("detail-fields").innerHTML = Object.entries(fields).length
      ? Object.entries(fields).map(([k, v]) =>
          `<div class="field-chip"><div class="fk">${esc(k)}</div><div class="fv">${esc(String(v))}</div></div>`
        ).join("")
      : `<span class="muted">No structured fields.</span>`;
    $("detail-tooltip").innerHTML = rec.tooltip_html || "<em>No tooltip HTML.</em>";
    $("detail-panel").classList.remove("hidden");
    $("detail-panel").scrollIntoView({ behavior: "smooth", block: "nearest" });
  } catch (err) {
    addLog(`Detail load error: ${String(err)}`, "error");
  }
}

$("detail-close").addEventListener("click", () => $("detail-panel").classList.add("hidden"));

// ── Export ─────────────────────────────────────────────────────────────────────

async function doExportJson(kindFilter: string) {
  const path = await save({
    defaultPath: `turtle-${kindFilter || "all"}-export.json`,
    filters: [{ name: "JSON", extensions: ["json"] }],
  });
  if (!path) return;
  try {
    const n = await invoke<number>("export_json", { path });
    $("export-status").textContent = `✓ Exported ${fmtNum(n)} records → ${path}`;
    addLog(`JSON export: ${fmtNum(n)} records saved to ${path}`);
  } catch (err) {
    $("export-status").textContent = `Error: ${String(err)}`;
    addLog(`Export error: ${String(err)}`, "error");
  }
}

async function doExportCsv(kindFilter: string) {
  const path = await save({
    defaultPath: `turtle-${kindFilter || "all"}-export.csv`,
    filters: [{ name: "CSV", extensions: ["csv"] }],
  });
  if (!path) return;
  try {
    const n = await invoke<number>("export_csv", {
      path,
      kindFilter: kindFilter || null,
    });
    $("export-status").textContent = `✓ Exported ${fmtNum(n)} rows → ${path}`;
  } catch (err) {
    $("export-status").textContent = `Error: ${String(err)}`;
  }
}

async function doExportSql(kindFilter: string) {
  const path = await save({
    defaultPath: `turtle-${kindFilter || "all"}-export.sql`,
    filters: [{ name: "SQL", extensions: ["sql"] }],
  });
  if (!path) return;
  try {
    const n = await invoke<number>("export_sql", {
      path,
      kindFilter: kindFilter || null,
    });
    $("export-status").textContent = `✓ Exported ${fmtNum(n)} INSERTs → ${path}`;
  } catch (err) {
    $("export-status").textContent = `Error: ${String(err)}`;
  }
}

async function doExportDb() {
  const path = await save({
    defaultPath: "turtle.db",
    filters: [{ name: "SQLite DB", extensions: ["db"] }],
  });
  if (!path) return;
  try {
    const srcPath = (await invoke<{ path: string }>("db_stats")).path;
    await copyFile(srcPath, path);
    $("export-status").textContent = `✓ Copied database → ${path}`;
  } catch (err) {
    $("export-status").textContent = `Error: ${String(err)}`;
  }
}

$("exp-json-all").addEventListener("click", () =>
  doExportJson(($("exp-json-kind") as HTMLSelectElement).value)
);
$("exp-csv-all").addEventListener("click", () =>
  doExportCsv(($("exp-csv-kind") as HTMLSelectElement).value)
);
$("exp-sql-all").addEventListener("click", () =>
  doExportSql(($("exp-sql-kind") as HTMLSelectElement).value)
);
$("exp-db").addEventListener("click", doExportDb);

// ── Import ─────────────────────────────────────────────────────────────────────

$("import-btn").addEventListener("click", async () => {
  const selected = await openDialog({
    filters: [{ name: "JSON", extensions: ["json"] }],
    multiple: false,
  });
  if (!selected) return;
  const path = Array.isArray(selected) ? selected[0] : selected;
  $("import-status").textContent = "Importing…";
  try {
    const result = await invoke<{ inserted: number; updated: number; skipped: number }>(
      "import_json", { path }
    );
    $("import-status").textContent =
      `✓ Done — inserted: ${fmtNum(result.inserted)}, updated: ${fmtNum(result.updated)}, skipped: ${fmtNum(result.skipped)}`;
    addLog(`Import complete: +${fmtNum(result.inserted)} new, ~${fmtNum(result.updated)} updated`);
    refreshDbPill();
  } catch (err) {
    $("import-status").textContent = `Error: ${String(err)}`;
    addLog(`Import error: ${String(err)}`, "error");
  }
});

$("find-gaps-btn").addEventListener("click", async () => {
  const start = Number(($("enrich-start") as HTMLInputElement).value);
  const end   = Number(($("enrich-end")   as HTMLInputElement).value);
  $("gaps-status").textContent = "Scanning…";
  try {
    const gaps = await invoke<GapEntry[]>("gap_ids", {
      kinds: ALL_KINDS,
      startId: start,
      endId: end,
    });
    pendingGaps = gaps;
    $("gaps-status").textContent = `Found ${fmtNum(gaps.length)} missing IDs.`;
    renderGapsList(gaps);
    ($("scrape-gaps-btn") as HTMLButtonElement).disabled = gaps.length === 0;
  } catch (err) {
    $("gaps-status").textContent = `Error: ${String(err)}`;
  }
});

$("scrape-gaps-btn").addEventListener("click", async () => {
  if (!pendingGaps.length) return;
  const cfg = readConfig();
  try {
    activateTab("scrape");
    setRunning(true);
    addLog(`Targeted scrape: ${fmtNum(pendingGaps.length)} missing IDs`);
    await invoke("start_targeted_scrape", { gaps: pendingGaps, cfg });
  } catch (err) {
    setRunning(false);
    addLog(`Targeted scrape error: ${String(err)}`, "error");
  }
});

function renderGapsList(gaps: GapEntry[]) {
  const MAX_CHIPS = 200;
  const list = $("gaps-list");
  list.innerHTML = gaps.slice(0, MAX_CHIPS).map(g =>
    `<span class="gap-chip">${esc(g.kind)}#${g.entry}</span>`
  ).join("") + (gaps.length > MAX_CHIPS
    ? `<span class="muted" style="font-size:.75rem;align-self:center"> …and ${fmtNum(gaps.length - MAX_CHIPS)} more</span>`
    : "");
}

// ── Profiles ───────────────────────────────────────────────────────────────────

async function refreshProfiles() {
  try {
    const profiles = await invoke<ScrapeProfile[]>("load_profiles");
    const list = $("profiles-list");
    $("no-profiles").classList.toggle("hidden", profiles.length > 0);
    list.innerHTML = profiles.map(p => `
      <div class="profile-row">
        <span class="pname">${esc(p.name)}</span>
        <span class="pmeta">
          ${esc(p.cfg.kinds.join(", "))} · IDs ${p.cfg.start_id}–${p.cfg.end_id}
          · conc ${p.cfg.concurrency} · ${p.cfg.rate_per_sec} r/s
        </span>
        <button class="btn-sec btn-load-profile" data-name="${esc(p.name)}" style="margin-left:auto">Load</button>
        <button class="btn-ghost btn-del-profile" data-name="${esc(p.name)}">✕</button>
      </div>`).join("");

    list.querySelectorAll<HTMLButtonElement>(".btn-load-profile").forEach(btn => {
      btn.addEventListener("click", () => loadProfile(btn.dataset.name!, profiles));
    });
    list.querySelectorAll<HTMLButtonElement>(".btn-del-profile").forEach(btn => {
      btn.addEventListener("click", async () => {
        await invoke("delete_profile", { name: btn.dataset.name });
        refreshProfiles();
      });
    });
  } catch (_) { /* ignore */ }
}

function loadProfile(name: string, profiles: ScrapeProfile[]) {
  const p = profiles.find(x => x.name === name);
  if (!p) return;
  const cfg = p.cfg;
  document.querySelectorAll<HTMLInputElement>('input[name="kind"]').forEach(cb => {
    cb.checked = cfg.kinds.includes(cb.value as Kind);
  });
  ($("start") as HTMLInputElement).value = String(cfg.start_id);
  ($("end")   as HTMLInputElement).value = String(cfg.end_id);
  ($("conc")  as HTMLInputElement).value = String(cfg.concurrency);
  $("conc-out").textContent = String(cfg.concurrency);
  ($("rate")  as HTMLInputElement).value = String(cfg.rate_per_sec);
  $("rate-out").textContent = String(cfg.rate_per_sec);
  ($("use-sitemap") as HTMLInputElement).checked = cfg.use_sitemap;
  activateTab("scrape");
  addLog(`Profile "${name}" loaded.`);
}

// ── Stats ──────────────────────────────────────────────────────────────────────

async function refreshStats() {
  try {
    const s = await invoke<RichStats>("rich_stats");
    $("stats-total").textContent = fmtNum(s.total);
    const max = Math.max(...Object.values(s.by_kind), 1);
    $("kind-bars").innerHTML = ALL_KINDS.map(k => {
      const n = s.by_kind[k] ?? 0;
      const pct = (n / max) * 100;
      return `<div class="kind-bar-row">
        <span class="muted">${k}</span>
        <div class="kind-bar-track"><div class="kind-bar-fill" style="width:${pct.toFixed(1)}%"></div></div>
        <span class="muted" style="text-align:right">${fmtNum(n)}</span>
      </div>`;
    }).join("");
    $("stats-rows").innerHTML = ALL_KINDS.map(k => `
      <tr>
        <td>${k}</td>
        <td>${fmtNum(s.by_kind[k] ?? 0)}</td>
        <td>${fmtNum(s.skipped_by_kind[k] ?? 0)}</td>
      </tr>`).join("");
    $("stats-meta").innerHTML = [
      `DB path: ${esc(s.path)}`,
      `DB size: ${fmtBytes(s.db_size_bytes)}`,
      `Oldest fetch: ${s.oldest_fetch ? tsToDate(s.oldest_fetch) : "—"}`,
      `Newest fetch: ${s.newest_fetch ? tsToDate(s.newest_fetch) : "—"}`,
      `Skipped total: ${fmtNum(s.skipped_total)}`,
    ].join(" · ");
  } catch (_) { /* ignore if DB not ready */ }
}

async function refreshDbPill() {
  try {
    const s = await invoke<{ total: number }>("db_stats");
    $("dbpill").textContent = fmtNum(s.total);
  } catch (_) { /* ignore */ }
}

$("refresh-stats-btn").addEventListener("click", refreshStats);

$("clear-skipped-btn").addEventListener("click", async () => {
  if (!confirm("Clear all _skipped entries? This allows re-scraping every previously skipped ID.")) return;
  try {
    const n = await invoke<number>("clear_skipped", { kinds: ALL_KINDS });
    addLog(`Cleared ${fmtNum(n)} skipped entries.`);
    refreshStats();
  } catch (err) {
    addLog(`Clear skipped error: ${String(err)}`, "error");
  }
});

// Switch to Stats tab → auto-refresh
tabBtns.find(b => b.dataset.tab === "stats")?.addEventListener("click", refreshStats);
tabBtns.find(b => b.dataset.tab === "profiles")?.addEventListener("click", refreshProfiles);

// ── Keyboard shortcuts ─────────────────────────────────────────────────────────
// Ctrl+1..6 → tab switch
document.addEventListener("keydown", (e) => {
  if (!e.ctrlKey) return;
  const map: Record<string, string> = {
    "1": "scrape", "2": "browse", "3": "export",
    "4": "import", "5": "profiles", "6": "stats",
  };
  if (map[e.key]) { e.preventDefault(); activateTab(map[e.key]); }
  // Ctrl+F → focus search
  if (e.key === "f") { e.preventDefault(); activateTab("browse"); $("q").focus(); }
});

// ── Init ───────────────────────────────────────────────────────────────────────

refreshDbPill();
```

✅ Verify: `npm run build` is clean (zero TypeScript errors).

---

### 14.4 — Update capabilities for `fs:allow-read-file`

The import feature reads a user-selected file via `fs::read()`. Add the read permission to `src-tauri/capabilities/default.json`:

Replace the permissions array content:

```json
"permissions": [
  "core:default",
  "core:event:default",
  "core:window:default",
  "shell:default",
  "shell:allow-open",
  "dialog:default",
  "fs:default",
  "fs:allow-read-file",
  "fs:allow-write-text-file",
  "fs:allow-write-file",
  "fs:allow-copy-file"
]
```

✅ Verify: `npm run tauri dev` — app launches, all 6 tabs present, Scrape tab shows config + live progress area.

---

### 14.5 — Commit

```powershell
cd turtle-scraper
git add -A
git commit -m "step 14: premium UI — 6 tabs, ETA, RPS chart, CSV/SQL export, import/enrich, profiles, detail view, theme"
```

---

## Troubleshooting cheat sheet

| Symptom | Fix |
|---|---|
| `linker rust-lld not found` | Install "Desktop development with C++" in Visual Studio Build Tools |
| WebView2 missing | Install Edge WebView2 Runtime (auto on Win11) |
| 403 storm | Pause → solve in browser → Resume; reduce rate to 4 r/s |
| `database is locked` | We use WAL — close external sqlite browsers |
| `cargo check` fails on `governor` | `cargo update -p governor` |
| Tauri capability errors | Re-check `capabilities/default.json` matches §3.2 + §14.4 |
| Empty search after scrape | Verify `db_stats` total > 0; check LIKE uses `%` |
| App window blank | Run `npm run dev` first; check `devUrl` in `tauri.conf.json` |
| `fs:allow-copy-file` unknown | Update `@tauri-apps/cli` to latest `^2` and re-run `npm install` |
| Import silently skips all rows | JSON must be the array format produced by this app's own export |
| Gap scan returns 0 gaps | All IDs in range either have records or are in `_skipped`; use "Clear all skipped" in Stats then re-scan |

