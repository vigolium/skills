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
- [Step 1.5 — Sweep, when the scope is a host list](#step-15--sweep-when-the-scope-is-a-host-list)
- [Step 2 — Scan and gate](#step-2--scan-and-gate)
- [Watching a scan while it runs](#watching-a-scan-while-it-runs)
- [Step 3 — Read the results](#step-3--read-the-results)
- [Step 4 — Confirm a finding](#step-4--confirm-a-finding)
- [Step 5 — Hand off](#step-5--hand-off)
- [Bulk replay](#bulk-replay)
- [Payload fuzzing](#payload-fuzzing)
- [Reading exports without a database](#reading-exports-without-a-database)
- [Filesystem tree output](#filesystem-tree-output)
- [Agentic scans](#agentic-scans)
- [The `-j` envelope](#the--j-envelope)
- [Exit codes](#exit-codes)
- [Gotchas](#gotchas)

---

## Mental model

The four load-bearing facts — **the database is the state** (scans write,
queries read, commands compose through the DB not pipes), **three machine
contracts** (`-j/--json` = one compact envelope for triage; `--format
jsonl`/`export` = bulk full-fidelity stream for archival; `--events ndjson` =
a live stream while a scan runs), **non-interactive by default** (TUI is
`--tui`, destructive needs `--force`), and **every JSON summary hands you the
next command** (the `query` field) — are stated in full in `SKILL.md`, always in
context. This file assumes them and drills into each step.

Scoping: `--project-uuid <uuid>`, `--project-name <name>`,
`VIGOLIUM_PROJECT_UUID=<uuid>`, or `VIGOLIUM_PROJECT_NAME=<name>`.

## Token discipline

Under `--json`, `finding` and `traffic` keep headers and high-signal metadata but
**bound** bodies so a scan can't blow your context window:

- Bodies are previewed (request ~1 KiB, response ~2 KiB) with `body_size`,
  `body_sha256`, and `body_truncated:true` so you know more exists.
- Binary/static bodies (images, fonts, JS bundles, gzip) are stubbed as
  `{"body_omitted":"binary", …}`.
- gzip is decoded transparently, but **only the first 1 MiB** — see the trap
  below before you trust a decoded body.
- Findings get a ±240-char `response_evidence` snippet windowed on the match
  instead of the whole page.

**A gzip body over 1 MiB is capped, and says so.** The JSON decoder holds at most
1 MiB, so a larger body comes back as a prefix — but the metadata describes the
*whole* body, which is what lets you act on it:

| Field | Meaning |
|---|---|
| `body_size` | size of the **complete** body, not of the prefix you got |
| `body_truncated` | `true` whenever `body` is shorter than `body_size` (set under `--full-body` too) |
| `body_sha256` | digest of the **complete** body — safe as a dedupe key |
| `decoder_capped` | the 1 MiB decoder limit was hit; `--full-body` cannot lift it |
| `decoder_hint` | names the command that returns the whole body |

So `decoder_capped: true` means **do not conclude a string is absent** from this
body. Pull it whole with the filesystem exporter, which has no such cap:

```bash
vigolium db export --format fs -o out --uuid "$RECORD_UUID"
# → out-traffic/<host>/<id>.resp.body   (complete, gzip-decoded)
```

Control it:

| Flag | Effect | Commands |
|------|--------|----------|
| `--compact` | metadata only, drop bodies — best for surveys. Also drops `request`/`response` entirely, so there is **no header access** under it | finding, traffic, db ls |
| `--fields a,b,c` | project the JSON to just these top-level keys (cuts tokens hardest). An unknown name is a **usage error listing the valid set**, not a silent drop | finding, traffic, db ls |
| `--full-body` | complete decoded bodies, except past the 1 MiB gzip cap — which is flagged `decoder_capped` | finding, traffic, db ls |
| `--with-records` | embed the linked HTTP records → self-contained triage bundle. Capped at 20 per finding | finding |
| `--record-limit N` | change that cap (`0` = no cap). Over the cap you get `records_total` + `records_truncated: true` | finding |
| `--record-fields a,b` | project the **nested** records independently of `--fields` | finding |
| `--min-severity` | threshold that expands upward (`high` → high + critical) | finding |
| `--agentic-scan <uuid>` | every finding from an agent run (expands to the whole run tree) | finding |
| `--pick N` | keep only the 1-based position(s) from the result list — `2`, `1,3`, `2-4` | finding |
| `--markdown` | render as Markdown (evidence + fenced `http` blocks) instead of JSON | finding, traffic |
| `--raw` | full raw HTTP request/response, human format | finding, traffic |
| `--group-by <field>` | **count** the matched records by one field instead of listing them; `--group-limit N` bounds the buckets (default 20, `0` = all) | traffic |

Rule of thumb: **survey with `--compact --fields`, then drill with `--id` +
`--with-records`.** Never fetch full bodies for more than one record at a time.

And before either: if the question is a *shape* rather than a row set, count in
SQL. `--group-by` applies the same filters the listing would, so the buckets
describe exactly the rows `traffic` would have shown:

```bash
vigolium traffic -j --group-by status_code --host target.example
# → items: [{value,count}…] plus group_by, total_records, other_groups, other_records
```

`total` is the number of **buckets**; `total_records` is how many records were
counted. The tail past `--group-limit` is reported as `other_groups` /
`other_records` — a capped grouping never looks complete. Groupable fields:
`host`, `method`, `status_code`, `response_content_type`, `source`, `scan_uuid`,
`ip`, `is_authenticated` (an unknown name is a usage error listing the set).

`--fields` composes safely with `--with-records`: the projection narrows the
finding's own keys and never removes the `records` array, because that was
requested by a different flag. `--record-fields` narrows the nested rows.

To discover the selectable names, ask for a bad one — the error lists the whole
supported set for that view:

```bash
vigolium traffic -j --fields '?' 2>&1 | head -3
```

Note that JSON field names are not SQLite column names: the JSON key is `host`,
the storage column is `hostname`. Go by the error's list, not by the schema.

## Step 1 — Scope

```bash
# What's already in the DB?
vigolium db stats --json

# Is the environment ready (binaries, providers, templates)?
vigolium doctor --json

# What modules exist / match a topic?
vigolium module ls xss
```

## Step 1.5 — Sweep, when the scope is a host list

With more than a handful of targets, probe first and scan what's worth scanning.
`run probe` sends one request per target — no discovery, no fuzzing — and streams
JSONL on stdout (progress goes to stderr, so `2>/dev/null` or `--silent` gives a
clean pipe):

```bash
vigolium run probe -T hosts.txt --json --no-response -c 50 2>/dev/null > sweep.jsonl

# Alive + interesting, ranked by attack surface:
jq -r 'select(.type=="http_record") | .data
       | select(.status_code < 500)
       | [.surface_score, .status_code, .url, .response_title] | @tsv' sweep.jsonl \
  | sort -rn | head -40
```

Each record carries `status_code`, `response_time_ms`, `response_title`,
`response_words`, `surface_score` (0-100, percent of the surface signals present), `ip`, and the full DNS
answer (`a`/`aaaa`/`cname`). Add `--tls-probe` for the certificate (`tls` object:
version, cipher, subject/SANs, issuer, validity, fingerprints). Field names match
httpx's, so an existing consumer reads them unmodified.

Then scan only the survivors:

```bash
jq -r 'select(.type=="http_record") | .data | select(.surface_score >= 30) | .url' \
  sweep.jsonl > worth-scanning.txt
vigolium scan -T worth-scanning.txt --fail-on high
```

Caveat: `a`/`aaaa`/`cname`/`tls` are **output-only** — re-reading the database
later shows `ip` alone. Keep the JSONL, or re-probe.

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

**Which command?** Use `scan-url` / `scan-request` only when you're confirming
**one specific request** — full params, a deep path, or custom headers you need
sent verbatim. For a bare host/root URL, `scan -t` expands the surface for you.

A full `scan -t` can run well past 30 min (discovery defaults to 1h, spidering
30m, no total cap) — **launch it in the background** and poll the DB, or bound it:

```bash
vigolium scan -t https://target.example --scanning-max-duration 30m --fail-on high
```

For a quick single-phase pass, run it directly: `vigolium run spidering -t <url>`.

## Watching a scan while it runs

Don't poll the DB and don't invent a patience message. Ask for the stream:

```bash
vigolium scan -t https://target.example --events ndjson 2>/dev/null | jq -c .
```

One JSON object per line on **stdout**, flushed per event; the human console
stays on stderr, so `2>/dev/null` yields clean NDJSON with **zero** non-JSON
lines. Available on `scan`, `run`, `scan-url`, `scan-request`.

```jsonc
{"v":1,"ts":"…","scan_uuid":"…","type":"scan.started","target":"https://…","strategy":"lite","phases":["heuristics-check","discovery","dynamic-assessment"],"db_path":"/tmp/…","pace":{"rate_limit":20,"concurrency":10,"max_per_host":10}}
{"v":1,"ts":"…","scan_uuid":"…","type":"phase.started","phase":"discovery"}
{"v":1,"ts":"…","scan_uuid":"…","type":"phase.progress","phase":"discovery","requests_sent":1204,"findings":3}
{"v":1,"ts":"…","scan_uuid":"…","type":"waf.block","host":"app.example.com","vendor":"cloudflare","status_code":403,"detail":"edge is filtering scan traffic — results for this host may be incomplete"}
{"v":1,"ts":"…","scan_uuid":"…","type":"waf.pacing","host":"app.example.com","vendor":"akamai","concurrency_from":40,"concurrency_to":8}
{"v":1,"ts":"…","scan_uuid":"…","type":"finding.new","severity":"high","confidence":"firm","module_id":"sqli-error-based","url":"…"}
{"v":1,"ts":"…","scan_uuid":"…","type":"phase.finished","phase":"discovery","status":"completed","duration_ms":184000,"requests_sent":1544,"findings":3}
{"v":1,"ts":"…","scan_uuid":"…","type":"error","phase":"spidering","message":"chromium launch failed"}
{"v":1,"ts":"…","scan_uuid":"…","type":"scan.finished","status":"completed","duration_ms":903000,"findings_by_severity":{"high":3,"medium":11},"records_written":4820}
```

What each type is for:

| Type | Read it for |
|---|---|
| `scan.started` | The canonical `phases[]` the aliases resolved to, the `db_path` to query afterwards, and the `pace` that actually applied (plus `phase_pace` when a phase differs). |
| `phase.progress` | Liveness. `requests_sent` every 5s is the real slope — a crawl that is working versus one that is wedged. |
| `waf.block` | The edge started filtering. Results for that host are **incomplete** — do not read a thin surface as a clean target. |
| `waf.pacing` | Vigolium slowed itself down on purpose. Attribute the slowdown here, not to the target. |
| `finding.new` | Metadata only. Evidence lives in `db_path`; query it with `finding -j --with-records` afterwards. |
| `error` | A phase failed and the scan **carried on** — non-fatal by construction. A failure that ends the run rides on `scan.finished{status:"failed"}` instead, so this is never a reason to abandon a run that is still producing findings. |
| `scan.finished` | Terminal. `status`, `findings_by_severity`, `records_written`. |

Four contracts you can build on:

- **Every line carries `scan_uuid`.** A sweep is several `vigolium scan`
  invocations; this is how you attribute an event to one.
- **`v` is the event-schema version.** Gate on it.
- **`scan.finished` is always last**, including `status:"interrupted"` on
  SIGINT/SIGTERM.
- **Its absence means the process was killed outright** (SIGKILL can't be
  caught). Treat a stream that stops without a terminal event as a hard kill,
  not as a completed scan.

This replaces scraping `vigolium log` for `[waf-block-detected]` /
`[waf-pacing-armed]` markers — every notice that reaches the log also reaches the
stream, with no `log` invocation and no session-listing walk.

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

**Pin a session DB with `$VIGOLIUM_DB_PATH`** to avoid repeating `--db` on every
command. The open order is `--db` → `$VIGOLIUM_DB_PATH` → `database.sqlite.path`
in config → the built-in default. Setting it also makes **read** commands
(`finding`/`traffic`/`fuzz -u`) treat that file as a stateless source — project
scoping off, same as `-S --db` above — while writes still go to it. It never
turns a *scan* stateless (that would collide with `--db`).

```bash
export VIGOLIUM_DB_PATH=~/scans/acme.sqlite
vigolium finding --min-severity high      # reads acme.sqlite, scoping off
vigolium scan -t https://acme.example     # writes into acme.sqlite
```

The scoping-off flip is conditional: it applies once the pinned file **exists and
is a usable vigolium store**. Run that `finding` before the first scan of the
session and the read stays project-scoped against a store it just created — which
is the right behavior (it prints an empty table instead of erroring), but means
`project_scoped` is the field to check, not something to assume. A pin that can
never *become* a database — a directory, or a parent dir that cannot be created —
is a hard error rather than a silent fall-through to the shared default; a path
that merely does not exist yet is fine.

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

## The `-j` envelope

Every `-j/--json` command emits the **same shape**, so one parser handles them
all — including `log ls`, `ingest`, and `db ls --list-tables`/`--list-columns`.
Parse `items`; do not write key fallbacks.

Two shape notes that are not exceptions to the envelope, just to `items`:

- `db stats -j` puts a summary **object** in `items`, not a row array — don't run
  a universal `.items[]` over it.
- `ingest -j` has an empty `items` (`[]`); its payload is the sibling fields
  `records_ingested`, `input_format`, `record_source`, `duration_ms`.

**Errors are JSON too.** A failed `-j` command writes a parseable error object to
**stdout** and the human `✖ Error:` line to stderr, so a consumer never has to
scrape prose:

```jsonc
{ "schema_version": 1, "command": "traffic", "ok": false,
  "error": { "code": "source_missing", "message": "…", "exit_code": 1 } }
```

`error.code` is the stable branch point: `usage_error`, `source_missing`,
`source_unreadable`, `source_incompatible`, `gate_tripped`, `failed`. A
successful envelope has no `error` key. Still check the exit code — a parseable
error is still an error.

The three source codes exist because they are three different next moves, and
they used to arrive as one indistinguishable failure:

| `error.code` | What the file is | What to do |
|---|---|---|
| `source_missing` | nothing at that path (**needs `--read-only`** — see below) | fix the path |
| `source_unreadable` | not a database, or corrupt | treat it as damaged evidence; do not "recover" it by reading somewhere else |
| `source_incompatible` | **valid SQLite that is not a vigolium store** — it opens, passes an integrity check, and every read fails on a missing table (**needs `--read-only`**) | you are pointed at the wrong file (a zero-byte stub, another tool's database) |
| *(no error, `total: 0`)* | a real vigolium store with no matching rows — **or**, without `--read-only`, a path that was wrong | nothing has scanned this yet — go scan it, after confirming the path |

The last row is the one worth guarding, and it is wider than it looks. A store
that was never a vigolium database and a target nobody has scanned are opposite
facts, and reporting the first as "no traffic found" turns a wrong path into a
thin attack surface. By default the two are genuinely indistinguishable: a read
against a missing path *creates* an empty store there and returns `total: 0`,
exit 0. Pass **`--read-only`** on reads that must hit an existing store and the
two source codes above start firing; see the exit-code table for the full matrix.

```jsonc
{
  "schema_version": 1,
  "command": "traffic",
  "project_uuid": "…",
  "project_scoped": true,
  "db_path": "/home/me/.vigolium/database-vgnm.sqlite",
  "total": 39,
  "offset": 0,
  "limit": 100,
  "items": [ … ],
  "query": "vigolium traffic --uuid <uuid> --json --full-body",
  "generated_at": "2026-09-04T10:11:12.345Z",
  "generated_at_ms": 1788453072345
}
```

| Field | Why it is there |
|---|---|
| `schema_version` | Gate on it. Bumped on any breaking field change. |
| `items` | **The** row array, and the only one. The pre-envelope names (`records`, `findings`, `scans`, `rows`, `stats`) are gone from the default output: they duplicated every row on the wire, which on a 20-record compact read was 19,983 bytes against 7,884 of actual rows. `--json-legacy-keys` (or `VIGOLIUM_JSON_LEGACY_KEYS=1`) brings the alias back for a caller still migrating, at that cost. |
| `db_path` | The database this command actually opened. Assert it; the open order ends at one shared default file, so a fall-through silently mixes engagements. |
| `project_uuid` | The project that *would* apply — **not** proof a project filter was applied. Under `-S` scoping is off and the result spans every project, yet this field still names one. Treat it as advisory; `total` against the row count is the real check. |
| `query` | A suggested follow-up. **Read it before running it** — see below. |
| `generated_at` / `_ms` | RFC3339 with **exactly 3** fractional digits, plus an epoch-millisecond sibling. Both are safe to compare against a JS `toISOString()`; the old microsecond form sorted `…785113Z` *before* `…785Z`. |

### The `query` hint

Every hint is a valid, **read-only** command — running one never puts traffic on
the wire. `traffic`/`db ls` hand back `traffic --uuid <uuid> --json --full-body`
(inspect the record), `finding` hands back `finding --id <int> --json
--with-records` (the evidence bundle), `ingest` hands back a plain recency
listing.

To re-send a request you must ask for it explicitly — `replay -u <uuid>` or
`fuzz -u <uuid>`. Those are never suggested by a read.

Check the contract once at startup:

```bash
vigolium version --json   # {version, commit, schema_version, db_schema_version}
```

## Exit codes

| Code | Meaning |
|-----:|---------|
| `0` | success (or `--soft-fail` forced it) |
| `1` | error — the command failed to do its job |
| `2` | usage error — bad flag, bad value, rejected combination |
| `3` | `fuzz --fail-on-match` matched |
| `4` | `--fail-on <sev>` gate tripped |

**`4` is not a failure.** The scan ran to completion, wrote its output, and found
something at or above the threshold — the opposite outcome from `1`, where it
never got that far. Branch on them separately; treating every non-zero as
breakage either ignores real outages or reports every finding as one.

On the read commands:

| Read | Exit | `error.code` |
|---|---:|---|
| `--fields bogus` | `2` | `usage_error` (lists the valid names) |
| `--db <not a sqlite file>` | `1` | `source_unreadable` |
| `--db <missing file>` | **`0`** | **none** — the file is *created*, `{"total":0,"items":[]}` |
| `--db <missing file> --read-only` | `1` | `source_missing` |
| `--db <sqlite, but not a vigolium store>` | **`0`** | **none** — vigolium's tables are created *inside it*, `{"total":0,"items":[]}` |
| `--db <foreign sqlite, unwritable> --read-only` | `1` | `source_incompatible` |
| `finding --id <a UUID>` | `2` | `usage_error` (names the flag that reads that namespace) |
| `traffic --group-by <unknown field>` | `2` | `usage_error` (lists the groupable fields) |
| `--id 999999` (no such finding) | `0` | —, `{"total":0,"items":[]}` |
| `--search zzzznomatch` (genuine zero hits) | `0` | —, `{"total":0,"items":[]}` |

**The bolded rows are the trap.** Opening a database writes to it — that is what
lets `scan --db ~/new.sqlite` work on a fresh path — so a *read* against a wrong
path does not fail. It creates the store, finds nothing in it, and reports
`total: 0` with exit 0, indistinguishable from a target nobody has scanned. The
foreign-store case is worse: vigolium creates its own tables inside another
tool's SQLite file.

`source_missing` and `source_incompatible` therefore only reach you when the open
is prevented from writing. **Add `--read-only` to any read whose store is
supposed to already exist** — it is the flag that converts both silent zeroes
into the error you wanted, and it costs nothing on a store that is really there.
`source_unreadable` needs no such help: a non-SQLite file fails either way.

Failing that, assert the envelope's `db_path`, and check `total` against a second
query you know matches — the same technique that separates "`--id 999999` does
not exist" from "`--search` legitimately matched nothing", which are also one
result.

Always check the exit code before parsing — and never `2>&1` into a JSON parser:
the machine object is on stdout, the human line on stderr, and merging them
corrupts the stream.

`--fail-on <info\|suspect\|low\|medium\|high\|critical>` on `scan`/`run`/
`scan-url`/`scan-request` exits `4` when a finding at or above that severity is
present. **Output is always written first** — the gate only changes the exit
code. `--soft-fail` (global) forces exit 0 even on error and overrides
`--fail-on`. Under `-P/--parallel` the gate is evaluated **per child**; the
parent batch fails only when every target fails.

## Gotchas

- `-S` means `--stateless` **everywhere**, and is accepted as a harmless no-op on
  commands where it has no meaning — no per-command acceptance table needed. The
  one holdout is deprecated: on `server`/`ingest` it is still an alias for
  `--scan-on-receive` and warns. Use the long `--scan-on-receive` there.
- `$VIGOLIUM_DB_PATH` pointing at a path that can never hold a database (a
  directory, an uncreatable parent) is a hard error, not a silent fall-through to
  the shared default. A path that simply does not exist yet is created. Read
  `db_path` off any `-j` envelope to confirm which store you actually got.
- `vigolium log <uuid> | cat` terminates promptly even when the scan row still
  says `running` (a deadline or SIGKILL leaves it that way forever). Auto-follow
  is off for a non-TTY stdout and for a stale row; pass `--follow` to force it.
- `--json` (one compact object) ≠ `--format jsonl` (bulk, one line per row).
- **Fetch one record by UUID with `traffic --uuid <uuid>`** (also on `db ls`,
  repeatable/comma-separated). It is an exact equality match applied **before**
  pagination, so `-n`/`--offset` can never hide a match. A UUID passed as the
  *positional* term still matches 0 rows — that searches URL/path/body, not
  identity, so use the flag.
- **`-S`/`--stateless` is a scoping mode, not a read-only one.** By default,
  opening any database writes to it: `mkdir -p` on the parent, a `journal_mode`
  PRAGMA (a header rewrite) and a WAL checkpoint. Pass **`--read-only`** when the
  source is evidence — it skips all three, leaves the file's SHA-256 unchanged,
  creates no sidecars, reads a `chmod 444` file fine, and errors on a missing
  path instead of creating an empty database there. It is accepted on the read
  commands and refused (exit 2) on anything that writes.
- `db export` validates `--format`, `-o` and the date range **before** opening
  the destination, so a rejected run leaves an existing file intact.
- `db export --uuid` works on every format including `--format fs`, and is
  applied in SQL before paging, so a record outside the current page is found
  rather than reported missing.
- `replay` has no `--mutate` — payload fuzzing lives entirely in `vigolium fuzz`.
- `--with-browser` produces **no diff** — a navigation has no status code or
  body to compare, so `result` is null and a `browser` object is emitted instead.
- Agentic scans need a configured LLM provider (`agent.olium` in
  `vigolium-configs.yaml`); check with `vigolium doctor --json`.
- `db clean` with no selector is rejected — it never implicitly wipes the DB.
