# Server & Ingestion Reference

> **Related:** [agent-loop.md](agent-loop.md) for `--mirror-fs` consumption and bulk replay

Complete flag reference for `server`, `ingest`, and `traffic` commands.

## Table of Contents

- [server](#server)
- [ingest](#ingest)
- [traffic](#traffic)
- [traffic --replay](#traffic---replay) — see `references/agent-loop.md`

---

## server

**Usage:** `vigolium server [flags]`

Start the API server with Swagger UI, ingestion endpoints, and optional scan-on-receive mode.

### server-specific flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--alternative-ingest-key` | — | []string | — | Additional API key for ingestion endpoints (repeatable) |
| `--burp-bridge-url` | `-B` | string | `$VIGOLIUM_BURP_BRIDGE_URL` | Merge live Burp traffic from this loopback bridge URL into `/api/http-records` |
| `--catchup-threads` | — | int | `4` | **Deprecated: no-op** (catch-up scanning is disabled; the live scan-on-receive scanner covers post-cursor records) |
| `--demo-only` | — | bool | `false` | Expose only the demo allowlist (GET `/api/findings`, `/api/http-records`, `/api/modules`, `/api/stats`, `/api/extensions`) |
| `--disable-catchup` | — | bool | `false` | **Deprecated: no-op** (catch-up scanning is already disabled) |
| `--disable-warm-session` | — | bool | `false` | Disable agent warm session pooling |
| `--export-ca` | — | string | — | Write the ingest-proxy MITM CA certificate to this path and exit |
| `--host` | — | string | `0.0.0.0` | Bind address for the API server |
| `--ingest-proxy-port` | — | int | `0` (disabled) | Transparent HTTP proxy port for recording traffic |
| `--mem-buffer` | — | int | `10000` | In-memory queue capacity before spilling to disk |
| `--mirror-fs` | — | string | — | Mirror ingested traffic + findings to a live flat file tree under this dir (`<dir>/traffic`, `<dir>/findings`), in addition to the DB (config `server.mirror_fs_path`) |
| `--no-agent` | — | bool | `false` | Disable all agent endpoints and warm session pooling |
| `--no-auth` | `-A` | bool | `false` | Run server without API key authentication |
| `--no-swagger` | — | bool | `false` | Disable the Swagger UI and API spec endpoint |
| `--output` | `-o` | string | — | Write findings to specified output file |
| `--passive-only` | — | bool | `false` | With `-S`/`--scan-on-receive`, run passive modules only (no active scan traffic; includes secret detection) |
| `--proxy-insecure` | — | bool | `false` | When intercepting HTTPS (`--proxy-mitm`), skip verification of the upstream server's TLS certificate |
| `--proxy-mitm` | — | bool | `false` | Intercept HTTPS through `--ingest-proxy-port` using a generated CA so TLS traffic is recorded (trust the CA printed at startup) |
| `--service-port` | — | int | `9002` | Port for the REST API server |
| `--timeout` | — | duration | `15s` | HTTP request timeout for background scan workers (e.g. `30s`, `1m`) |
| `--view-only` | — | bool | `false` | Run server in read-only mode (disables scanning, ingestion, agent, and all write endpoints) |

### Server Authentication

API key resolution priority (highest to lowest):
1. `--no-auth` / `-A` flag — disables auth entirely
2. `--alternative-ingest-key` flag
3. `VIGOLIUM_API_KEY` environment variable
4. `server.auth_api_key` in config file

### Key Global Flags for Server

| Flag | Description |
|------|-------------|
| `-t <url>` | Target URL (used with `-S` for scope) |
| `-S` / `--scan-on-receive` | Auto-scan every ingested request |
| `--full-native-scan-on-receive` | Run the full native pipeline (discovery + spidering + dynamic-assessment) on received records, not dynamic-assessment only |
| `-c` / `--concurrency` | Worker pool size |
| `--proxy` | Proxy for outgoing requests |

(`--disable-fetch-response` is an **ingest-only** flag — it is not registered on `server`.)

### Examples

```bash
# Basic server
vigolium server

# Custom port, no auth
vigolium server --service-port 8443 --no-auth

# With scan-on-receive
vigolium server -t https://example.com --scan-on-receive

# With transparent proxy
vigolium server --ingest-proxy-port 8080

# High concurrency server
vigolium server -c 200 --mem-buffer 50000

# Mirror ingested traffic + findings to a live browsable file tree
vigolium server --ingest-proxy-port 8080 --mirror-fs ./mirror
```

### Live Filesystem Mirror (`--mirror-fs`)

`--mirror-fs <dir>` (config `server.mirror_fs_path`) mirrors every saved HTTP record and finding to `<dir>/traffic/` + `<dir>/findings/` as they are persisted — in addition to the database — so an external agent can read ingested Burp/proxy traffic as files in real time (`ls`/`grep`/`jq`).

- **Same layout as `--format fs`**: per-host subdirs with `0001.req` (a leading `@target <scheme>://<authority>` line then the raw request), `0001.resp.headers`, `0001.resp.body` (gzip-decoded), and `0001.md` for findings cross-linked to their `.req`.
- **Append-only index**: writes `<root>/traffic/index.jsonl` + `<root>/findings/index.jsonl` (one JSON object per line), vs the one-shot export's single `index.json` array.
- **Non-blocking**: a background goroutine handles all disk I/O and never blocks the DB save path (the buffer drops jobs with a warning if it overflows — the database is unaffected).
- **Resumes across restarts**: per-host id numbering continues from the highest existing `.req`/`.md` file.
- **Server-ingestion-only**: wired via the repository's `OnRecordSaved`/`OnFindingSaved` callbacks, which fire only on genuinely new inserts (deduplicated saves do not). CLI scans and other repo users are unaffected. Setup is best-effort — a failure logs a warning and the server continues without it.

### REST API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/ingest` | Submit HTTP records for ingestion |
| `POST` | `/api/agent/run/query` | Single-shot agent prompt execution |
| `POST` | `/api/agent/run/autopilot` | Autonomous AI-driven scanning session |
| `POST` | `/api/agent/run/swarm` | AI-guided targeted / full-scope scan |
| `POST` | `/api/agent/run/audit` | Whitebox audit (`driver: auto\|both\|audit\|piolium`) |
| `POST` | `/api/agent/chat/completions` | OpenAI-compatible chat completions |
| `GET` | `/api/agent/status/list` | List agent runs |
| `GET` | `/api/agent/status/:id` | Check agent run status |
| `GET` | `/api/agent/sessions[/:id[/logs\|/artifacts]]` | List / inspect agent sessions, logs, and artifacts |
| `GET` | `/` | Swagger UI dashboard |

---

## ingest

**Usage:** `vigolium ingest [flags]`

Ingest HTTP requests into the database, either locally or via a remote server.

### ingest-specific flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--server` | `-s` | string | — | Server URL for remote ingestion (omit for local mode) |

### Key Global Flags for Ingest

| Flag | Description |
|------|-------------|
| `-t <url>` | Base URL / target for the ingested data |
| `-i <file>` | Input file path |
| `-I <format>` | Input format (urls, openapi, burp, curl, har, etc.) |
| `-S` | After ingesting, scan the records (local mode only) |
| `--spec-url` | Use server URLs from OpenAPI spec |
| `--spec-header` | HTTP headers for OpenAPI requests |
| `--spec-var` | OpenAPI parameter values as key=value |
| `--spec-default` | Default value for required parameters (default: `1`) |
| `--disable-fetch-response` | Store request-only (don't fetch responses) |
| `--scope-origin` | Origin scope mode for filtering |
| `--no-tech-filter` | Disable the tech-stack allowlist (run every module regardless of detected stack; auto-on at `--intensity deep`) |
| `--no-waf-pacing` | Disable proactive CDN/WAF-edge pacing (reactive back-off after a WAF block still applies) |

### Local vs Remote Mode

- **Local mode** (default): Ingests directly into the local SQLite database, fetches HTTP responses
- **Remote mode** (`--server <url>`): Sends records to a running vigolium server via API
- `--scan-on-receive` is ignored in remote mode (server handles scanning)

### Examples

```bash
# Local ingest from OpenAPI spec
vigolium ingest -t https://api.example.com -I openapi -i spec.yaml

# Local ingest from Burp export
vigolium ingest -t https://example.com -I burp -i export.xml

# Pipe URLs from stdin
cat urls.txt | vigolium ingest -i -

# Ingest + auto-scan
vigolium ingest -t https://example.com -I openapi -i spec.yaml -S

# Remote ingest to server
vigolium ingest -s http://localhost:9002 -I openapi -i spec.yaml

# Request-only (no response fetching)
vigolium ingest -t https://example.com -I burp -i export.xml --disable-fetch-response
```

---

## traffic

**Usage:** `vigolium traffic [search-term] [flags]`

**Aliases:** `traffics`, `tf`

Browse stored HTTP traffic. Shortcut for `vigolium db ls http_records`.

### Filter flags (persistent, inherited by replay)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--host` | string | — | Filter by hostname pattern (wildcard supported) |
| `--method` | []string | — | Filter by HTTP method (repeatable, e.g. --method GET --method POST) |
| `--status` | []int | — | Filter by HTTP status code (repeatable, e.g. --status 200 --status 404) |
| `--path` | string | — | Filter by URL path pattern |
| `--from` | string | — | Show records after this date (YYYY-MM-DD or RFC3339) |
| `--to` | string | — | Show records before this date (YYYY-MM-DD or RFC3339) |
| `--search` | string | — | Fuzzy search across URLs, paths, and hostnames |
| `--header` | string | — | Search within HTTP header names and values |
| `--body` | string | — | Search within HTTP request/response body content |
| `--source` | string | — | Filter by record source (e.g. scanner, ingest-cli, ingest-server, ingest-proxy, seed) |
| `--sort` | string | `created_at` | Sort field: uuid, created_at, sent_at, method, status, time |
| `--asc` | bool | `false` | Sort in ascending order (default: descending) |
| `--limit` / `-n` | int | `100` | Maximum records to display |
| `--offset` | int | `0` | Number of records to skip (for pagination) |

### Display flags (traffic only)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--tree` | bool | `false` | Display as host/path hierarchy tree |
| `--raw` | bool | `false` | Full raw HTTP request and response |
| `--burp` | bool | `false` | Burp Suite-style colored format |
| `--markdown` | bool | `false` | Render matched records as Markdown (request/response in fenced http blocks) to stdout |
| `--columns` | []string | — | Columns to show (comma-separated, e.g. HOST,METHOD,PATH,STATUS) |
| `--exclude-columns` | []string | — | Columns to hide (comma-separated) |
| `--exclude-search` / `--exclude-header` / `--exclude-body` | []string / string | — | Inverse-search filters (drop records where the term appears) |
| `--stateless` / `-S` + `--glob-db` | bool / string | — | Read from a `.jsonl`/`.sqlite` export (or a glob merged into one temp DB) with project scoping off |

With `-j`/`--json`, `traffic` emits **one compact, token-aware object** (headers kept, bodies preview-capped, binary/static stubbed) built for coding-agent consumption. Shape it with `--compact` (metadata only), `--fields a,b,c` (project top-level keys), or `--full-body` (complete bodies) — the same contract as `finding`/`db ls`. See SKILL.md recipe 14c.

### Available Columns

UUID, HOST, METHOD, PATH, STATUS, TIME, SIZE, WORDS, CONTENT_TYPE, SENT_AT, TITLE, AUTH, STATUS_PHRASE, REQ_HEADERS, RESP_HEADERS, SOURCE, REMARKS

Default columns: HOST, METHOD, PATH, STATUS, CONTENT_TYPE, SIZE, WORDS, TIME, TITLE, SOURCE

### Argument Routing

- `vigolium traffic` — default table view
- `vigolium traffic <term>` — fuzzy search
- `vigolium traffic tree` — tree view
- `vigolium traffic list` or `ls` — default table view

### Examples

```bash
# Browse all traffic
vigolium traffic

# Fuzzy search
vigolium traffic login
vigolium traffic api/v2

# Tree view
vigolium traffic --tree

# Burp-style output
vigolium traffic --burp

# Filter by host and method
vigolium traffic --host api.example.com --method POST,PUT

# Filter by status code
vigolium traffic --status 200,301

# Date range
vigolium traffic --from 2024-01-01 --to 2024-06-30

# Custom columns
vigolium traffic --columns HOST,METHOD,PATH,STATUS,AUTH
```

---

## traffic --replay

Re-sending stored traffic — both `traffic --replay` (human comparison table,
`--with-browser`) and the top-level `vigolium replay` (structured JSONL diffs) —
is documented with the rest of the confirm/hand-off workflow in
**`references/agent-loop.md`** → *Bulk replay*.

Burp-bridge **listing** sync flags stay here because they operate on the
`traffic` listing, not on replay:

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--burp-bridge-url` / `-B` | string | `$VIGOLIUM_BURP_BRIDGE_URL` | Merge live traffic from this loopback Burp bridge URL with local DB records |
| `--save-to-vigolium-db` | bool | `false` | Persist the live Burp records selected by the active filters into the database (requires `--burp-bridge-url`; not with `--replay`) |
| `--save-to-burp` | bool | `false` | Copy the DB records selected by the active filters into Burp's Target site map (requires `--burp-bridge-url`; not with `--replay`) |

The two `--save-to-*` flags are mutually exclusive with `--replay`.
