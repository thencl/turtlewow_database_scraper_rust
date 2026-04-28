# API REFERENCE — turtlecraft.gg + Rate Limits

> Source of truth for every URL, selector, and rate cap the scraper uses.
> If the live site changes, edit this file first, then update the parser.

---

## 1. Endpoints

The site is an AoWoW fork. Every detail page is plain server-rendered HTML; no auth, no CSRF, GET only.

| Kind   | Detail URL                                       | ID range (1.18.1 build) |
|--------|--------------------------------------------------|-------------------------|
| Item   | `https://database.turtlecraft.gg/?item={id}`     | `1 .. 90000`            |
| Spell  | `https://database.turtlecraft.gg/?spell={id}`    | `1 .. 60000`            |
| Quest  | `https://database.turtlecraft.gg/?quest={id}`    | `1 .. 40000`            |
| NPC    | `https://database.turtlecraft.gg/?npc={id}`      | `1 .. 90000`            |
| Object | `https://database.turtlecraft.gg/?object={id}`   | `1 .. 50000`            |

A page for a non-existent ID returns HTTP 200 with the string `Tried to view a non-existing` somewhere in the body — the parser uses that as the "skip" sentinel so we do not waste a DB row.

### Sitemap shortcut (preferred over brute-force)

`https://database.turtlecraft.gg/sitemap.xml` lists every existing entity. The agent **must** fetch it first and only enqueue IDs that appear in the sitemap. This cuts the request volume by ~80 %.

If the sitemap 404s, fall back to brute-forcing the ID ranges above.

---

## 2. Rate limits — non-negotiable

These numbers were chosen to stay below the site's Cloudflare WAF rule (~30 req/s/IP, 1000 req/min/IP).

| Setting                     | Value           | Where                       |
|-----------------------------|-----------------|-----------------------------|
| Max concurrent requests     | **8**           | `tokio::sync::Semaphore`    |
| Global rate                 | **12 req / s**  | `governor::RateLimiter`     |
| Per-host burst              | **20**          | `governor::Quota::burst()`  |
| Per-request timeout         | **20 s**        | `reqwest` builder           |
| Retry attempts              | **5**           | custom backoff loop         |
| Backoff schedule            | 1, 2, 4, 8, 16 s (+ jitter ±20 %) |       |
| 429 handling                | honor `Retry-After`, otherwise 30 s |   |
| 403 + cf-mitigated:challenge| pause queue, raise UI event |               |
| Daily cap (safety net)      | **250 000 requests** | counted in DB table `_meta` |

If the user explicitly says "go faster", the agent may double concurrency to **16** and the rate to **24 req/s**, but never higher without written user consent.

---

## 3. CSS selectors (parser contract)

Tested against pages on 2026-04-26. Every selector below resolves to exactly one element on a valid page.

### Common (all kinds)

| Field      | Selector                                  | Notes                                 |
|------------|-------------------------------------------|---------------------------------------|
| `name`     | `h1.text` *first child text node*         | Strip leading icons.                  |
| `tooltip`  | `table.infobox + table` (the right column)| Full HTML kept as `tooltip_html`.     |
| `category` | `div.text > a:nth-child(2)`               | E.g. "Armor", "Weapon".               |
| `not_found`| body contains `Tried to view a non-existing` | If true → skip.                    |

### Item-specific

| Field        | Selector                            |
|--------------|-------------------------------------|
| `quality`    | `h1.text` `class` attr → `q0..q5`   |
| `item_level` | `td:contains("Item Level") + td`    |
| `required_level` | `td:contains("Requires Level") + td` |
| `bind`       | `td:contains("Binds")`              |
| `slot`       | `table.infobox td.q0`               |
| `icon`       | `div.iconlarge ins` `style` → url() |

### Spell-specific

| Field        | Selector                            |
|--------------|-------------------------------------|
| `school`     | `td:contains("School")`             |
| `cast_time`  | `td:contains("Cast time")`          |
| `cooldown`   | `td:contains("Cooldown")`           |
| `range`      | `td:contains("Range")`              |
| `description`| `div.spelldesc`                     |

### Quest-specific

| Field        | Selector                            |
|--------------|-------------------------------------|
| `level`      | `td:contains("Level")`              |
| `min_level`  | `td:contains("Required level")`     |
| `objectives` | `h2:contains("Description") + p`    |
| `rewards`    | `div#rewards`                       |
| `text`       | `div#qtxt-desc`                     |

### NPC-specific

| Field   | Selector                       |
|---------|--------------------------------|
| `level` | `td:contains("Level")`         |
| `react` | `td:contains("React")`         |
| `type`  | `td:contains("Type")`          |
| `loot`  | `div#tab-drops table`          |

### Object-specific

| Field   | Selector                       |
|---------|--------------------------------|
| `type`  | `td:contains("Type")`          |
| `loot`  | `div#tab-loot table`           |

---

## 4. Headers the client must send

```
User-Agent:      TurtleScraper/1.0 (+local archive)
Accept:          text/html,application/xhtml+xml
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection:      keep-alive
Referer:         https://database.turtlecraft.gg/
```

`Accept-Encoding` is set automatically by `reqwest` when the `gzip` and `brotli` features are on.

---

## 5. Cloudflare challenge handling

If `cf-mitigated: challenge` is received, the backend sets the `challenge` atomic flag and emits a
`progress` event with `state: "challenge"`. The scraper queue pauses automatically.

**`tauri::WebviewWindow::cookies()` does not exist in Tauri 2** — do not attempt to implement
programmatic cookie handoff. It will not compile.

**Correct procedure:**
1. The UI shows the challenge banner and a "Open database.turtlecraft.gg" button.
2. The user clicks it — `tauri-plugin-shell` opens the URL in the system's default browser.
3. The user solves the Turnstile challenge once. Cloudflare marks that IP as trusted.
4. The user returns to the app and clicks **Resume**. The `reqwest` client continues using the
   same network identity (same IP) — no cookie transfer needed.
5. If challenges persist, reduce `rate_per_sec` to 3–4 and retry.

The `clear_skipped` command can be used afterwards to re-attempt any IDs that errored during the
challenge window.

---

## 6. Useful Turtle WoW reference (read-only context)

- Custom in-game APIs (not used by this scraper, but listed for completeness): <https://turtle-wow.fandom.com/wiki/Category:API_Functions>
- Server build notes: <https://turtle-wow.org/news>

This scraper does **not** call the in-game API. It only reads the public website.
