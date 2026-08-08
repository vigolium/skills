# Server Reference

> **Related:** [ingest.md](ingest.md) for feeding traffic into the server ·
> [agent-loop.md](agent-loop.md) for `--mirror-fs` consumption and bulk replay ·
> [burp.md](burp.md) for the Burp bridge · [data.md](data.md) for browsing what landed

Complete flag reference for `vigolium server` — the REST API server with Swagger
UI, ingestion endpoints, an optional recording proxy, and optional
scan-on-receive mode.

To *push* traffic into a running server (or ingest locally without a server), see
[ingest.md](ingest.md).

## Table of Contents

- [server](#server)
- [Authentication](#server-authentication)
- [Key global flags](#key-global-flags-for-server)
- [Recording proxy (`--ingest-proxy-port`)](#recording-proxy)
- [HTTPS interception (`--proxy-mitm`)](#https-interception)
- [Live filesystem mirror (`--mirror-fs`)](#live-filesystem-mirror---mirror-fs)
- [REST API endpoints](#rest-api-endpoints)
- [Examples](#examples)

---

## server

**Usage:** `vigolium server [flags]`

Start the API server. By default it exposes the Swagger UI, the `/api/ingest`
endpoint, and the agent endpoints, all behind API-key auth. It does **not** scan
anything until you either call an endpoint or pass `-S`/`--scan-on-receive`.

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

(`--disable-fetch-response` is an **ingest-only** flag — it is not registered on
`server`. See [ingest.md](ingest.md).)

### Server Authentication

API key resolution priority (highest to lowest):
1. `--no-auth` / `-A` flag — disables auth entirely
2. `--alternative-ingest-key` flag
3. `VIGOLIUM_API_KEY` environment variable
4. `server.auth_api_key` in config file

`--alternative-ingest-key` is repeatable, so you can hand different feeders
(a Burp extension, a CI job, a proxy client) their own key without sharing the
primary one:

```bash
vigolium server \
  --alternative-ingest-key ci-runner-key \
  --alternative-ingest-key burp-plugin-key
```

### Key Global Flags for Server

| Flag | Description |
|------|-------------|
| `-t <url>` | Target URL (used with `-S` for scope) |
| `-S` / `--scan-on-receive` | Auto-scan every ingested request |
| `--full-native-scan-on-receive` | Run the full native pipeline (discovery + spidering + dynamic-assessment) on received records, not dynamic-assessment only |
| `-c` / `--concurrency` | Worker pool size |
| `--proxy` | Proxy for outgoing requests |

`-S`/`--scan-on-receive` is the **same letter** as `--stateless` elsewhere but a
different flag — on `server`/`ingest` it means *scan-on-receive*. Pair it with
`-t <url>` so the auto-scan has a scope; without a scope it scans every host it
receives.

---

## Recording proxy

`--ingest-proxy-port <port>` starts a transparent HTTP proxy. Point a browser or
tool at it and every request/response it sees is stored as an HTTP record (source
`ingest-proxy`), ready to scan or browse:

```bash
# Terminal 1 — capture.
vigolium server --ingest-proxy-port 8080

# Terminal 2 — drive traffic through it, then scan what was captured.
curl -x http://127.0.0.1:8080 http://target.example/
vigolium scan --only dynamic-assessment -t http://target.example
```

For an all-in-one capture-and-scan, add `-S -t <url>` so each request is scanned
as it arrives.

## HTTPS interception

Plain `--ingest-proxy-port` records HTTP only; HTTPS passes through opaquely. Add
`--proxy-mitm` to decrypt TLS with a generated CA (trust the CA printed at
startup, or export it up front):

```bash
# Write the CA out so you can trust it in the client first.
vigolium server --export-ca ./vigolium-ca.pem

# Then run the intercepting proxy.
vigolium server --ingest-proxy-port 8080 --proxy-mitm

# Skip upstream TLS verification (self-signed / staging origins).
vigolium server --ingest-proxy-port 8080 --proxy-mitm --proxy-insecure
```

## Live Filesystem Mirror (`--mirror-fs`)

`--mirror-fs <dir>` (config `server.mirror_fs_path`) mirrors every saved HTTP
record and finding to `<dir>/traffic/` + `<dir>/findings/` as they are persisted
— in addition to the database — so an external agent can read ingested
Burp/proxy traffic as files in real time (`ls`/`grep`/`jq`).

- **Same layout as `--format fs`**: per-host subdirs with `0001.req` (a leading `@target <scheme>://<authority>` line then the raw request), `0001.resp.headers`, `0001.resp.body` (gzip-decoded), and `0001.md` for findings cross-linked to their `.req`. Full layout + consumption walkthrough: [agent-loop.md → Filesystem tree output](agent-loop.md#filesystem-tree-output).
- **Append-only index**: writes `<root>/traffic/index.jsonl` + `<root>/findings/index.jsonl` (one JSON object per line), vs the one-shot export's single `index.json` array — so you can `tail -f` it while traffic lands.
- **Non-blocking**: a background goroutine handles all disk I/O and never blocks the DB save path (the buffer drops jobs with a warning if it overflows — the database is unaffected).
- **Resumes across restarts**: per-host id numbering continues from the highest existing `.req`/`.md` file.
- **Server-ingestion-only**: wired via the repository's `OnRecordSaved`/`OnFindingSaved` callbacks, which fire only on genuinely new inserts (deduplicated saves do not). CLI scans and other repo users are unaffected. Setup is best-effort — a failure logs a warning and the server continues without it.

## REST API Endpoints

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

All data endpoints are project-scoped via the `X-Project-UUID` header (mirrors
the CLI `--project` flag). Ingestion endpoints additionally accept any
`--alternative-ingest-key`.

## Examples

```bash
# Basic server (Swagger UI at http://localhost:9002/, API-key auth on).
vigolium server

# Custom port, no auth (dev only).
vigolium server --service-port 8443 --no-auth

# Scan every ingested request against a fixed scope.
vigolium server -t https://example.com --scan-on-receive

# Passive-only scan-on-receive (no active traffic; secret detection still runs).
vigolium server -t https://example.com -S --passive-only

# Full native pipeline (discovery + spidering + assessment) on each record.
vigolium server -t https://example.com -S --full-native-scan-on-receive

# Transparent recording proxy.
vigolium server --ingest-proxy-port 8080

# Intercepting HTTPS proxy (records TLS traffic under a generated CA).
vigolium server --ingest-proxy-port 8080 --proxy-mitm

# High-throughput ingestion server.
vigolium server -c 200 --mem-buffer 50000

# Mirror ingested traffic + findings to a live browsable file tree.
vigolium server --ingest-proxy-port 8080 --mirror-fs ./mirror

# Read-only dashboard for sharing results (no scanning, ingestion, or writes).
vigolium server --view-only

# Public demo: only the read-only allowlist is exposed.
vigolium server --demo-only

# Merge live Burp Proxy history into /api/http-records.
vigolium server -B http://127.0.0.1:9009
```

To send records to a running server from another host or a CI job, use the
remote ingest client — see [ingest.md → Local vs Remote Mode](ingest.md#local-vs-remote-mode).
