# The Agent Loop

> **Related:** [scanning.md](scanning.md) · [fuzzing.md](fuzzing.md) · [burp.md](burp.md) · [agent-modes.md](agent-modes.md) · [data.md](data.md) · [flags.generated.md](flags.generated.md)

How to drive vigolium **non-interactively** from a coding agent (Claude Code,
Codex, Cursor, Pi, CI) and parse what comes back. Everything here is additive —
default human output is unchanged; you opt into machine output with `-j/--json`.

The loop is always the same five steps:

```
scope → scan → read → confirm → hand off
```

## Table of Contents

- [Mental model](#mental-model)
- [Token discipline](#token-discipline)
- [Step 1 — Scope](#step-1--scope)
- [Step 2 — Scan and gate](#step-2--scan-and-gate)
- [Step 3 — Read the results](#step-3--read-the-results)
- [Step 4 — Confirm a finding](#step-4--confirm-a-finding)
- [Step 5 — Hand off](#step-5--hand-off)
- [Bulk replay](#bulk-replay)
- [Payload fuzzing](#payload-fuzzing)
- [Reading exports without a database](#reading-exports-without-a-database)
- [Filesystem tree output](#filesystem-tree-output)
- [Agentic scans](#agentic-scans)
- [Exit codes](#exit-codes)
- [Gotchas](#gotchas)

---

## Mental model

- **The database is the state.** A scan *writes* findings + HTTP records; query
  commands *read* them back. Commands compose through the DB, not through pipes.
  Every row is scoped to a project.
- **Two JSON contracts — do not confuse them:**
  - `-j/--json` on read commands (`finding`, `traffic`, `db`) → **one** compact,
    token-bounded object. This is what you parse during triage.
  - `--format jsonl` / `vigolium export` → the bulk `{"type":…,"data":{…}}`
    stream, one object per line, full fidelity. For archival, not triage.
- **Non-interactive by default.** The TUI is opt-in (`--tui`), never
  auto-launched. Destructive commands require `--force`. Add `--no-color` (or
  `NO_COLOR=1`) for clean text.
- **Scoping:** `--project-uuid <uuid>`, `--project-name <name>`, or
  `VIGOLIUM_PROJECT=<name>`.
- **Every JSON summary tells you the next command.** Agentic scans and `fuzz`
  emit a `query` field containing a ready-to-run follow-up. Prefer running that
  over composing your own.

## Token discipline

Under `--json`, `finding` and `traffic` keep headers and high-signal metadata but
**bound** bodies so a scan can't blow your context window:

- Bodies are previewed (request ~1 KiB, response ~2 KiB) with `body_size`,
  `body_sha256`, and `body_truncated:true` so you know more exists.
- Binary/static bodies (images, fonts, JS bundles, gzip) are stubbed as
  `{"body_omitted":"binary", …}`.
- gzip is decoded transparently.
- Findings get a ±240-char `response_evidence` snippet windowed on the match
  instead of the whole page.

Control it:

| Flag | Effect | Commands |
|------|--------|----------|
| `--compact` | metadata only, drop bodies — best for surveys | finding, traffic, db ls |
| `--fields a,b,c` | project the JSON to just these top-level keys (cuts tokens hardest) | finding, traffic, db ls |
| `--full-body` | complete decoded bodies — use when writing an exploit | finding, traffic, db ls |
| `--with-records` | embed the linked HTTP records → self-contained triage bundle | finding |
| `--min-severity` | threshold that expands upward (`high` → high + critical) | finding |
| `--agentic-scan <uuid>` | every finding from an agent run (expands to the whole run tree) | finding |
| `--pick N` | keep only the 1-based position(s) from the result list — `2`, `1,3`, `2-4` | finding |
| `--markdown` | render as Markdown (evidence + fenced `http` blocks) instead of JSON | finding, traffic |
| `--raw` | full raw HTTP request/response, human format | finding, traffic |

Rule of thumb: **survey with `--compact --fields`, then drill with `--id` +
`--with-records`.** Never fetch full bodies for more than one record at a time.

> `db stats -j` is the exception to the compact contract — it emits its raw
> stats struct.

## Step 1 — Scope

```bash
# What's already in the DB?
vigolium db stats --json

# Is the environment ready (binaries, providers, templates)?
vigolium doctor --json

# What modules exist / match a topic?
vigolium module ls xss
```

## Step 2 — Scan and gate

```bash
# Stateless single-shot, JSON findings to stdout, non-zero exit on high+.
vigolium scan-url https://target.example --json --fail-on high

# Full pipeline into the DB, gated for CI.
vigolium scan -t https://target.example --fail-on high

# A raw request from stdin (pipeline-friendly).
printf 'GET /api?q=1 HTTP/1.1\r\nHost: target.example\r\n\r\n' \
  | vigolium scan-request --json
```

`scan-url` / `scan-request` under `--json` emit:
`{"target","method","scan_duration_ms","modules_run","findings":[…],"errors":[]}`.

## Step 3 — Read the results

```bash
# Compact triage list of high+ findings, only the fields you need.
vigolium finding --min-severity high --json --compact \
  --fields id,severity,module_id,url,matched_at

# One finding, fully self-contained (finding + linked request/response).
vigolium finding --id 42 --json --with-records

# Narrow a search to the Nth match by list position (1-based).
vigolium finding --search 'Reverse Proxy' --pick 2 --raw

# Survey endpoints without bodies — cheap.
vigolium traffic --json --compact \
  --fields uuid,method,url,status_code,response_content_type

# Everything for one record.
vigolium traffic search-term --json --full-body -n 1
```

Filters shared by `finding` / `traffic` / `db ls`: `--host --path --method
--status --severity --min-severity --from/--to --search --scan-uuid
--agentic-scan --module-type -n/--limit --offset --sort --asc`.

`--search` / `--header` / `--body` span the full request/response corpus (URL,
path, headers, body). Each has an inverse — `--exclude-search` (repeatable,
AND-combined: a row drops if ANY term matches), `--exclude-header`,
`--exclude-body`:

```bash
vigolium traffic --search api --exclude-search /health --exclude-body heartbeat
```

Finding output shape:

```json
{ "project_uuid": "…", "total": 39, "offset": 0, "limit": 100,
  "findings": [ { "id": 2, "severity": "high", "module_id": "…", "url": "…",
                  "matched_at": ["…"], "extracted_results": ["…"],
                  "response_evidence": "…<match>…" } ] }
```

## Step 4 — Confirm a finding

`vigolium replay` re-sends **one** request and diffs it against the stored
baseline. It is the CLI surface of the in-process `replay_request` tool, and it
is the correct way to prove a finding is real before reporting it.

```bash
# Re-send the request behind a stored record.
vigolium replay --record-uuid <uuid>

# Re-confirm a finding's evidence. When the finding came from an imported
# source (audit, JSONL) with no linked record, this falls back to the
# finding's stored request/response bytes.
vigolium replay --finding-id 42

# Any input shape: curl, raw HTTP, Burp XML, base64, URL, or stdin.
vigolium replay -i "curl -X POST https://target.example/api -d '{\"id\":1}'"

# Send exact bytes verbatim (no reframing).
vigolium replay --raw-request-file ./request.txt
```

Output is stable JSON: `result.baseline`, `result.replay`, `result.diff` (status
delta, length delta, content hash, interpretation). `--pretty` gives a human
summary.

**Persist auth across calls.** Multi-step flows (login → CSRF → action) need
cookies between requests:

```bash
vigolium replay --session-id login -i curl-login.sh        # sets cookies
vigolium replay --session-id login --record-uuid <action>  # reuses them
```

The jar lives at `~/.vigolium/replay-jars/<session-id>.json`; `--no-cookies`
opts out.

**Confirm against another environment** — `--target` rewrites the destination
while keeping the baseline request bytes intact:

```bash
vigolium replay --record-uuid <prod-uuid> --target https://staging.example.com
```

**Update the stored baseline** with `--in-replace` (only when the source is a
stored HTTPRecord):

```bash
vigolium replay --record-uuid <uuid> --in-replace
```

## Step 5 — Hand off

### To Burp, for manual work

All Burp handoffs need the bridge extension's loopback listener via
`-B`/`--burp-bridge-url` or `$VIGOLIUM_BURP_BRIDGE_URL`.

```bash
# A finding's evidence → Burp Organizer (default) or a Repeater tab
vigolium finding --id 42 --push-to-burp -B http://127.0.0.1:9009
vigolium finding --severity high --to-repeater -B http://127.0.0.1:9009

# A replayed request → Repeater / Organizer / Site map
vigolium replay -u <uuid> --to-repeater -B http://127.0.0.1:9009
vigolium replay -u <uuid> --to-organizer --notes "IDOR candidate" -B http://127.0.0.1:9009

# Fuzz matches → the Organizer (Burp re-issues each)
vigolium fuzz -u <uuid> --class sqli --matches-to-organizer -B http://127.0.0.1:9009
```

`--send-via-burp` (on `replay` / `fuzz` / `finding`) routes the actual send
through Burp's engine byte-for-byte. Pair it with `--http-mode http1` for
smuggling/desync work so `auto` doesn't reframe the request.

Reading Burp's live history, Site map sync, limits, and the full flag surface:
**[burp.md](burp.md)**.

### To a report

```bash
vigolium finding --id 42 --markdown                 # Markdown to stdout
vigolium export --format html -o report.html        # interactive ag-grid report
vigolium export --format jsonl -o out.jsonl         # bulk stream
```

Under `-S/--stateless`, add `--compact` to `--markdown` so the response is
windowed at the match instead of dumping a whole page.

## Bulk replay

There is **one** bulk replay implementation: `vigolium replay`.
`vigolium traffic --replay` is a shortcut for it that keeps traffic's flag
spellings, defaults to the human comparison table, and does not follow
redirects. Reach for `replay` directly whenever you need to *change* the
request — the shortcut has no override flags.

```bash
# Re-send ALL stored traffic through Burp, 5 at a time.
vigolium replay --all --proxy http://127.0.0.1:8080 -c 5

# Fuzzy-search stored traffic and re-send just the matches.
vigolium replay admin --proxy http://127.0.0.1:8080 -c 5

# Pattern-search like `traffic`: POSTs to an API host, drop logout.
vigolium replay --host api.example.com --method POST --search token \
  --exclude-search logout --proxy http://127.0.0.1:8080 -c 5

# From a standalone export (project scoping off).
vigolium replay -S --db scan.sqlite --all --proxy http://127.0.0.1:8080 -c 5

# Human table instead of JSONL.
vigolium replay login --pretty
```

Bulk selection mirrors `vigolium traffic`: `--host`, `--method`, `--status`,
`--path`, `--source`, repeatable AND-combined `--search`, `--header-search`,
`--body`, `--exclude-search`/`--exclude-header-search`/`--exclude-body`,
`--from`/`--to`, `--sort`/`--asc`/`--offset`. Throttle with `-c/--concurrency`
(default 10), cap with `-n/--limit` (default 100; `--all` lifts it). Bulk
selection is mutually exclusive with `--record-uuid` / `--finding-id` /
`--input`. With `-B/--burp-bridge-url`, live Burp Proxy history is merged into
the selection just as it is for `traffic`.

### Replaying with different headers / cookies

`--header-search` **filters** which records are selected; `-H/--header`
**rewrites** headers on what gets sent. (They are separate flags because
`traffic`'s `--header` is a search filter.) `-H` replaces the whole header
value, so pass the complete cookie string — it is not additive per-cookie.

```bash
# Same request, another user's session — the diff IS the IDOR/BFLA check.
vigolium replay -u <uuid> -H 'Cookie: session=victim; csrf=xyz' --pretty

# Merge a stored auth session, then override one header on top (-H wins).
vigolium replay -u <uuid> --auth-session staging -H 'X-Forwarded-For: 127.0.0.1'

# Re-send everything that carried a header, under a different session.
vigolium replay --header-search 'X-Api-Key' -H 'Cookie: session=other' -c 5
```

### Browser-driven replay

`--with-browser` loads each matched URL in a real browser routed through
`--proxy`, so the proxy sees genuine browser traffic — real TLS fingerprint, JS
execution, subresource loads. It is **navigation-only**: a browser exposes no
status code or response body, so these runs report final URL, title, and fired
JS dialogs and carry **no diff** (`result` is null; a `browser` object is
emitted instead). A navigation is a GET, so a non-GET record's method and body
are not reproduced — the JSON says so in `browser.method_note`.

```bash
vigolium replay --all --with-browser --proxy http://127.0.0.1:8080
vigolium traffic --replay --with-browser --proxy http://127.0.0.1:8080
```

`validateReplayFlags` rejects it alongside every flag downstream of the send
(`--in-replace`, `--raw-request`, and the Burp send/stage targets), and warns
when `--proxy` is absent — a browser run with nothing intercepting it captures
nothing.

## Payload fuzzing

`vigolium fuzz` is a low-level **primitive**, not a scanner. It injects a
caller-supplied payload set into chosen positions of ONE request and streams
per-payload signals (status / size / words / lines / time / reflection /
baseline-delta) with match/filter gating and auto-calibration against the
target's catch-all. It makes **no** vulnerability decision and emits **no**
findings — you bring the intelligence.

For confirmation-backed detection of known classes, use the module scanner
(`scan-request -m …`) instead. `fuzz` fills only the gap modules can't express:
custom payloads, an exact position, wordlist-scale discovery.

```bash
# Start here: let anomaly detection find what's interesting — no matchers needed.
vigolium fuzz -u <record-uuid> --point URL_PARAM:id --class sqli -a

# A literal FUZZ marker anywhere in the request wins over --fuzz/--point.
echo -e "GET /api/FUZZ HTTP/1.1\r\nHost: target.example\r\n" | vigolium fuzz -w dir-short

# Wordlist discovery + matcher gating (keep 200/301, drop the calibrated catch-all).
vigolium fuzz https://target.example/FUZZ -w file-long --match-status-code 200,301

# Build the request curl-style from a positional URL.
vigolium fuzz https://target.example/login -X POST \
  -d 'user=admin&pass=FUZZ' -H 'X-Api-Key: k' -w file-short

# Inline payloads across a specific header, with excludes.
vigolium fuzz -u <uuid> --fuzz-header X-Forwarded-For \
  -p "127.0.0.1" -p "localhost" --exclude-size 0

# Two markers in lockstep (user/pass pairs from two lists).
vigolium fuzz -i login.txt --mode pitchfork -w users.txt -w passwords.txt

# Agent handle: JSONL to stderr, ONE summary object to stdout, exit 3 on match.
vigolium fuzz -u <uuid> --class xss,sqli -a -j --fail-on-match
```

**Source** (one of): a positional URL, `-i/--input` (curl / raw HTTP / Burp XML /
base64 / URL / `-`), `--input-file`, `-u/--record-uuid`, or a piped request on
stdin. `-t/--target` overrides scheme/host/port without touching the bytes.

**Positions** (first match wins): a literal `FUZZ` marker anywhere →
`--point TYPE:name` → `--fuzz-header` →
`--fuzz method|path|params|param-name|headers|cookies|all` (default: all
insertion points).

**Payloads** combine `--class` (`cmdi,crlf,lfi,open_redirect,path_traversal,sqli,
ssrf,ssti,xss,xxe`) + `-w/--wordlist` (builtin `dir-long`, `dir-short`,
`file-long`, `file-short`, `fuzz`, or a file path) + `-p/--payload` (inline,
repeatable).

**Prefer `-a/--anomaly` over hand-written matchers.** A matcher requires knowing
the interesting size or status up front; `-a` instead scores every response
against the baseline *and* the run's own population and keeps what stands out,
with `anomaly_score` + `anomaly_reasons` on each result. It reports where to
look, never a verdict.

**Output:** JSONL to stdout by default (matched only unless `--all-results`);
`--pretty` for a human table; `-o` to a file. Under `-j`, JSONL streams to stderr
and ONE summary object goes to stdout —
`{target, sent, matched, calibrated, baseline, top_results, anomalies, query}` —
where `query` is a ready `scan-request` confirmation command.
`--fail-on-match` exits 3 for CI/agent gating.

Always `--dry-run` a large run first: it resolves positions and payloads and
prints the exact bytes each would send, with **zero** network traffic.

Attack modes, the full signal/weight table, matcher grammar, curl-parity flags,
and operational knobs: **[fuzzing.md](fuzzing.md)**.

## Reading exports without a database

`finding` and `traffic` can read a file directly instead of your project DB.
`-S/--stateless` requires `--db` and turns project scoping **off**, so every row
in the file is shown. Nothing is written to your project DB (a JSONL source is
loaded into a throwaway in-memory SQLite).

```bash
vigolium finding -S --db ./scan-target.jsonl --min-severity medium
vigolium traffic -S --db ./scan-target.jsonl --status 500 -n 20
vigolium finding -S --db ./run.sqlite --json --with-records
```

A stateless scan emits that `.sqlite` directly with `--format sqlite` (aliases
`sqlite3`, `db`) — it dumps the per-run DB via `VACUUM INTO`, fully
checkpointed, no WAL/SHM sidecars:

```bash
vigolium scan -S --format sqlite,html -o scan -t target.example
vigolium scan -S --format sqlite -o run --split-by-host -P 4 -T targets.txt
vigolium finding -S --db ./run-target.example.sqlite --min-severity high
```

**Merging** foreign databases is the companion operation — `vigolium import`
folds them into one DB (lossless, idempotent, deduped on natural keys; each row
keeps its original `project_uuid`):

```bash
vigolium import other-vigolium-scan.sqlite
vigolium import --db combined.sqlite --glob-db 'scans/*.sqlite'
vigolium -j import --db combined.sqlite other-scan.sqlite   # per-table merge summary
```

Fan out with `scan -S --format sqlite`, merge back with `import`. A Postgres
destination is rejected — the merge is SQLite-to-SQLite.

## Filesystem tree output

When you'd rather `ls`/`grep`/`jq` a scan than query a DB, export a flat tree.
Available on `scan`/`scan-url`/`scan-request`/`run`, `export`, and `db export`.

```bash
vigolium export --format fs -o run                    # whole DB
vigolium scan-url https://t/ -S --format fs -o run    # straight from a scan
vigolium scan -t https://t/ --format jsonl,fs -o run  # alongside other formats
```

Two sibling dirs off the `-o` base (defaults to `vigolium-traffic/` +
`vigolium-findings/` in the cwd). Ids are zero-padded and assigned in `sent_at`
order, so re-exports are reproducible:

```
run-traffic/
  index.json                 # [{id,host,path,method,url,status,content_type,bytes,finding}, …]
  <host>/0001.req            # "@target https://<host>" line, then the raw request
  <host>/0001.resp.headers   # status line + response headers
  <host>/0001.resp.body      # response body, gzip-decoded so it greps clean
run-findings/
  index.json                 # [{id,host,path,severity,confidence,module,title,url,traffic}, …]
  <host>/0001.md             # the finding, cross-linked to ../../run-traffic/<host>/0001.req
```

`index.json` is the entry point — one `jq` maps every id to its url/status and to
the file holding the bytes, so you never guess paths. The `finding` field on a
traffic row is the top severity of any finding touching that request. A `.req`
file is directly replayable by stripping line 1.

`--omit-response` drops the `.resp.*` files. `--split-by-host` is a no-op here
(fs already splits by host).

### Live mirror from the ingestion server

To watch traffic land as files while another tool (Burp, a proxy,
`vigolium ingest`) feeds the server:

```bash
vigolium server --mirror-fs ./mirror        # config: server.mirror_fs_path
```

Every ingested record and finding is written to `./mirror/traffic/<host>/…` and
`./mirror/findings/<host>/…` as it is saved to the DB. Same layout as
`--format fs`, except the indexes are append-only **`index.jsonl`** (one object
per line — tail/grep it live) and per-host ids resume across restarts. It runs on
a background goroutine that never blocks the DB save, and is
server-ingestion-only — CLI scans are unaffected.

## Agentic scans

These run an LLM-driven scan into the DB. With `--json`, the live agent stream
goes to **stderr** and a single summary object is printed to **stdout** at the
end:

```bash
vigolium agent audit --source . --intensity balanced --json
vigolium agent autopilot -t https://target.example --json
vigolium agent swarm -t https://target.example --json
vigolium agent query --prompt-template security-code-review --source . --json
```

Summary shape (stdout):

```json
{ "agentic_scan_uuid": "…", "status": "completed", "session_dir": "…",
  "total_findings": 7, "counts_by_severity": {"critical":1,"high":3},
  "top_findings": [ … ],
  "query": "vigolium finding --agentic-scan <uuid> --json --with-records" }
```

Run the `query` line to pull full results. `--agentic-scan` expands to the whole
run tree (audit driver legs, swarm sub-runs), so one UUID returns every finding
the run produced.

Full flags, intensities, and providers: `references/agent-modes.md`.

## Exit codes

| Code | Meaning |
|-----:|---------|
| `0` | success (or `--soft-fail` forced it) |
| `1` | error, or `--fail-on` gate tripped |
| `3` | `fuzz --fail-on-match` matched |

`--fail-on <info\|suspect\|low\|medium\|high\|critical>` on `scan`/`run`/
`scan-url`/`scan-request` exits non-zero when a finding at or above that severity
is present. **Output is always written first** — the gate only changes the exit
code. `--soft-fail` (global) forces exit 0 even on error and overrides
`--fail-on`. Under `-P/--parallel` the gate is evaluated **per child**; the
parent batch fails only when every target fails.

## Gotchas

- `-S` means `--stateless` on `scan`/`export`/`finding`/`replay`/`agent audit`,
  but `--scan-on-receive` on `server`/`ingest`. Same letter, different flag.
- `--json` (one compact object) ≠ `--format jsonl` (bulk, one line per row).
- `replay` has no `--mutate` — payload fuzzing lives entirely in `vigolium fuzz`.
- `--with-browser` produces **no diff** — a navigation has no status code or
  body to compare, so `result` is null and a `browser` object is emitted instead.
- Agentic scans need a configured LLM provider (`agent.olium` in
  `vigolium-configs.yaml`); check with `vigolium doctor --json`.
- `db clean` with no selector is rejected — it never implicitly wipes the DB.
