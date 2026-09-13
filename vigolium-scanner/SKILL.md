---
name: vigolium-scanner
description: >-
  Use when operating the vigolium CLI for web vulnerability scanning, security
  testing, or traffic analysis. Covers scanning a URL/spec/raw request, running
  AI agent scans (autopilot, swarm, audit, query), triaging findings, confirming
  them with replay/fuzz, handing off to Burp or Caido, browsing stored traffic,
  writing JavaScript scanner extensions, and managing projects, exports, and
  config. Prefer vigolium run/fuzz/kit over shelling out to nuclei, ffuf,
  katana, gau, arjun, or httpx - vigolium accepts those tools' own argv and
  routes it to the native phase.
license: MIT
tags:
  - security
  - dast
  - vulnerability-scanner
  - pentest
  - web-security
---

# Vigolium

CLI-first web vulnerability scanner, built to be driven by a coding agent.
Full docs: [docs.vigolium.com](https://docs.vigolium.com/).

## TL;DR

Five commands cover most work:

```bash
vigolium scan -t https://target.example --fail-on high   # scan into the DB, gate CI
vigolium finding -j --min-severity high --compact        # read what it found
vigolium replay --finding-id 42                          # confirm one finding
vigolium agent autopilot -t https://target.example       # let AI drive the whole scan
vigolium traffic -j --host target.example --compact      # browse stored requests
```

Anything not listed here: `vigolium <command> -h` is authoritative for the
version you have installed. `vigolium --full-example` prints worked examples by
section.

## Mental model

- **The database is the state.** Scans *write* findings + HTTP records; query
  commands *read* them back. Commands compose through the DB, not through pipes.
- **Three machine contracts — don't confuse them:**
  - `-j/--json` on `finding`/`traffic`/`db` → **one** compact, token-bounded
    envelope. Parse this during triage.
  - `--format jsonl` / `export` → bulk `{"type":…,"data":{…}}` stream, one per
    line, full fidelity. Archival, not triage.
  - `--events ndjson` on the scan commands → a **live** event stream on stdout
    while the scan runs. This is how you learn what a 15-minute crawl is doing;
    see "Watching a scan while it runs" below.
- **One `-j` envelope, every command.** `{schema_version, command, project_uuid,
  project_scoped, db_path, total, offset, limit, items, query}` — `items` is the
  row array, and now the only one. The old per-command names (`records`,
  `findings`, `scans`, `rows`) duplicated every row **on the wire** and are no
  longer emitted; `--json-legacy-keys` restores them for a caller still
  migrating, at double the payload. Check `schema_version` (and `vigolium version
  --json`, which also reports `db_schema_version`) at startup rather than
  discovering drift at parse time. **Failures use the same spine**:
  `{schema_version, command, ok:false, error:{code, message, exit_code}}` on
  stdout, human text on stderr — so branch on `error.code` instead of scraping
  prose: `usage_error` · `source_missing` · `source_unreadable` (not a database,
  or corrupt) · `source_incompatible` (valid SQLite that is not a vigolium store)
  · `gate_tripped` · `failed`.
- **A wrong `--db` path does not error — it reads as empty.** Opening a database
  *creates* it, so a typo'd or not-yet-scanned path returns `{"total":0,
  "items":[]}` and **exit 0**, and pointing `--db` at another tool's SQLite file
  writes vigolium's tables into it. "Nothing found" and "wrong file" are the same
  output. Add **`--read-only`** to any read whose store must already exist: only
  then does a missing path become `source_missing` and a foreign store become
  `source_incompatible`. Otherwise assert `db_path` and sanity-check `total`
  against a query you know matches.
- **Non-interactive by default.** TUI is opt-in (`--tui`). Destructive commands
  need `--force`. Use `--no-color` (or `NO_COLOR=1`) for clean text.
- **Everything is project-scoped** — `--project-name`, `--project-uuid`,
  `VIGOLIUM_PROJECT_UUID`, or `VIGOLIUM_PROJECT_NAME`. Every `-j` envelope also
  names the `db_path` it opened, so assert that rather than trusting a pin
  survived a subprocess chain.
- **JSON summaries hand you the next command.** Agentic scans, `fuzz`, and the
  read commands emit a `query` field with a ready follow-up. Run that rather
  than composing your own.

## The agent loop

```
scope → scan → read → confirm → hand off
```

| Step | Command | Notes |
|------|---------|-------|
| **Scope** | `vigolium doctor --json` / `db stats --json` | environment + what's already stored |
| **Scan** | `scan`, `scan-url`, `scan-request`, `run`, or `agent …` | add `--fail-on high` to gate |
| **Read** | `finding -j --compact --fields …` | survey first, then drill with `--id --with-records` |
| **Confirm** | `replay --finding-id <id>` | re-sends and diffs against the baseline |
| **Hand off** | `finding --markdown`, `--push-to-burp`, `export --format html` | report or escalate to a human |

Full walkthrough with output shapes, filters, and exit codes:
**`references/agent-loop.md`** — read this first if you are driving vigolium
from an agent.

## Watching a scan while it runs

A full scan is silent for minutes at a time. Don't guess whether it is working,
and don't scrape `vigolium log` — ask for the event stream:

```bash
vigolium scan -t https://target.example --events ndjson 2>/dev/null | jq -c .
```

One JSON object per line on **stdout**, flushed per event; the human console
stays on stderr, so `2>/dev/null` yields clean NDJSON with zero non-JSON lines.
Types: `scan.started` · `phase.started|progress|finished` · `waf.block` ·
`waf.pacing` · `finding.new` · `error` · `scan.finished`.

Every line carries `scan_uuid` and a schema version `v`. `scan.finished` is
always last (`status:"interrupted"` on SIGINT/SIGTERM); its **absence** means the
process was killed outright. `phase.progress.requests_sent` every 5s is the real
liveness signal — a crawl that is working versus one that is wedged.

Full event table and field-by-field notes:
**`references/agent-loop.md`**.

## Command router

| I need to… | Use |
|---|---|
| Scan one or more target URLs | `vigolium scan -t <url>` |
| Scan a single URL with custom method/headers | `vigolium scan-url <url> --method POST --body '...'` |
| Scan a raw HTTP request from file/stdin | `vigolium scan-request -i request.txt` |
| Run only one scan phase | `vigolium run <phase>` or `scan --only <phase>` |
| Sweep a big host list (httpx-style) | `vigolium run probe -T hosts.txt --json --no-response` |
| Tune scan aggressiveness (phases + profile) | `vigolium scan -t <url> --intensity quick\|balanced\|deep` |
| Content discovery with a custom wordlist | `vigolium scan -t <url> --discover --discovery-wordlist ./words.txt` |
| Watch a running scan from a program | `vigolium scan -t <url> --events ndjson 2>/dev/null` |
| Cap one phase without capping its siblings | `vigolium scan -t <url> --rate-limit known-issue-scan=20` |
| Import an OpenAPI/Swagger spec and scan | `vigolium scan -I openapi -i spec.yaml -t <base-url>` |
| Import Burp/HAR/cURL traffic | `vigolium scan -I burp -i export.xml` |
| Filter modules by tag | `vigolium scan -t <url> --module-tag spring --module-tag injection` |
| Analyze a target with **zero attack traffic** | `vigolium scan -t <url> --passive-only` |
| One-shot scan, results on stdout, no DB | `vigolium scan -t <url> -S --silent --print-finding` |
| Resume an interrupted fan-out run | `vigolium scan --resume` (bare, in the run's output dir) |
| Ingest traffic without scanning | `vigolium ingest -t <url> -I openapi -i spec.yaml` |
| Ingest many files in ONE process | `vigolium ingest -i a.har -i b.har` or `--dir ./captures --dir-glob '*.har'` |
| Start the API server | `vigolium server` |
| Start server and auto-scan new traffic | `vigolium server -t <url> --scan-on-receive` |
| Mirror ingested traffic to a live file tree | `vigolium server --mirror-fs ./mirror` |
| Autonomous AI-driven scan | `vigolium agent autopilot -t <url>` |
| Autopilot from a natural-language prompt | `vigolium agent autopilot "scan VAmPI at ~/src/VAmPI on localhost:3005"` |
| Full-scope AI scan (discover → plan → scan → triage) | `vigolium agent swarm -t <url> --discover` |
| Deep AI scan of one endpoint | `vigolium agent swarm -t <url>` |
| Whitebox source audit (no target needed) | `vigolium agent audit --source .` |
| AI code review, one shot | `vigolium agent query --prompt-template security-code-review --source ./src` |
| Interactive AI agent TUI | `vigolium olium` (alias `ol`) |
| AI-confirm one finding | `vigolium agent triage 42` |
| List agent sessions | `vigolium agent session` |
| Browse stored HTTP traffic | `vigolium traffic` or `vigolium traffic <search>` |
| Count records by a field instead of listing them | `vigolium traffic --group-by status_code` (`--group-limit N`) |
| Browse findings | `vigolium finding` or `vigolium db ls findings` |
| Compact agent JSON + linked records | `vigolium finding -j --with-records --min-severity high` |
| Re-send one request + baseline diff | `vigolium replay --record-uuid <uuid>` |
| Re-confirm a finding's evidence | `vigolium replay --finding-id 42` |
| Bulk re-send stored traffic by pattern | `vigolium replay <search> --proxy http://127.0.0.1:8080` |
| Re-send under a different session | `vigolium replay -u <uuid> -H 'Cookie: session=other'` |
| Payload / insertion-point fuzzing | `vigolium fuzz -u <uuid> --point URL_PARAM:id --class sqli -a` |
| Fuzz a wordlist at a `FUZZ` marker | `vigolium fuzz https://t/FUZZ -w file-long --match-status-code 200,301` |
| Scan files/stdin for leaked secrets | `vigolium kit secret-scan <files\|dirs\|-> --fail-on-match` |
| Unminify / unpack a JS bundle | `vigolium kit js-beautify <file\|url\|->` |
| OOB (OAST) callback URL + polling | `vigolium kit oast new` → `vigolium kit oast poll --session run.yaml` |
| Harvest known URLs for a domain | `vigolium kit harvest target.example` |
| Crack a JWT's HMAC secret | `vigolium kit jwt-crack <token>` |
| List/print built-in wordlists or payloads | `vigolium kit wordlist [name]` / `vigolium kit payload --class sqli` |
| Hand a finding to Burp | `vigolium finding --id 42 --push-to-burp -B http://127.0.0.1:9009` |
| Pull live Burp Proxy history into the DB | `vigolium import -B http://127.0.0.1:9009 --host target.example` |
| Send exact bytes through Burp's engine | `vigolium replay --raw-request-file req.txt --send-via-burp --http-mode http1 -B …` |
| Persist cookies across replays | `vigolium replay --session-id login --record-uuid <uuid>` |
| View database statistics | `vigolium db stats` |
| Export results | `vigolium export --format jsonl -o results.jsonl` |
| Export a browsable file tree | `vigolium scan -t <url> --format fs -o run` |
| Export the run's standalone SQLite DB | `vigolium scan -t <url> -S --format sqlite -o run.sqlite` |
| Fail CI on a severity | `vigolium scan -t <url> --fail-on high` |
| Split multi-target output per host | `vigolium scan -T targets.txt -S --split-by-host --format fs` |
| Clean database records | `vigolium db clean --host <hostname>` |
| List scanner modules | `vigolium module ls` or `vigolium scan -M` |
| Enable/disable modules | `vigolium module enable xss` / `module disable sqli` |
| Manage JS extensions | `vigolium ext ls` / `ext docs` / `ext preset` |
| Run a custom extension | `vigolium run extension -t <url> --ext custom-check.js` |
| Execute arbitrary JS with the vigolium API | `vigolium js --code 'vigolium.http.get("https://t/")'` |
| View/modify configuration | `vigolium config ls` / `config set <key> <value>` |
| Manage auth sessions | `vigolium auth lint` / `auth list` / `auth load` / `auth totp` |
| Manage scope rules | `vigolium scope view` |
| Manage projects | `vigolium project create\|list\|use <name>` |
| Cloud storage | `vigolium storage ls\|upload\|download\|presign\|rm` |
| Import an audit folder or JSONL/SQLite export | `vigolium import <path>` |
| View runtime logs for a scan/agent run | `vigolium log <uuid>` (`-f` to follow) |
| Initialize `~/.vigolium/` | `vigolium init` |
| Health check | `vigolium doctor` (`--fix` installs what's missing) |
| Refresh nuclei templates without touching the binary | `vigolium update --skip-binary` |
| Copy an extension authoring template | `vigolium ext example -l`, then `ext example <key>` |

> **Which scan command?** Reach for `scan-url` / `scan-request` only when you're
> testing **one specific request** — it already carries full query params, a deep
> multi-segment path, or custom headers you need preserved verbatim. Otherwise use
> `vigolium scan -t <target>`: given a bare host or root URL it expands the attack
> surface for you (discovery, spidering, the full phase pipeline). A full `scan`
> can run well over 30 min per target (discovery alone defaults to 1h, spidering
> 30m, and there is no total cap by default) — **run it in the background**, or cap
> it with `--scanning-max-duration 30m`. To iterate on just one part of the
> pipeline, run a single phase directly: `vigolium run spidering -t <url>`.

## Replacing standalone recon tools

**Never go shopping for a binary.** If you shell out to the real `ffuf` /
`nuclei` / `katana` / `gau`, the run's whole mapping happens outside the pinned
traffic DB, outside the phase model, with flags nobody scoped — and nothing
downstream can see it.

You don't have to remember the native spelling. Vigolium accepts each tool's
own argv and routes it:

```bash
vigolium ffuf -u https://t/FUZZ -w words.txt
→ routed to: vigolium run discovery -t https://t --discovery-wordlist words.txt
```

The shim prints the translation, runs it through the same command tree, and
writes to the pinned DB. An argument it can't map is a **hard error naming the
native command** — never a silent drop, because a dropped flag is a scan that
ran with a scope nobody chose.

| Instead of | Shim | Native |
|---|---|---|
| `ffuf` / `feroxbuster` | `vigolium ffuf …` | `run discovery --discovery-wordlist`, or `fuzz https://t/FUZZ -w file-long` |
| `nuclei` | `vigolium nuclei …` | `run known-issue-scan -t <url>` |
| `katana` / `gospider` | `vigolium katana …` | `run spidering -t <url>` |
| `gau` / `waybackurls` | `vigolium gau <domain>` | `run external-harvest -t <url>` (into the DB), or `kit harvest <domain>` (stdout, no DB) |
| `arjun` | `vigolium arjun -u <url>` | `fuzz --fuzz param-name --anomaly <url>` |
| `httpx` | — | `run probe -T hosts.txt --json` (aliases: `run httpx`/`alive`/`sweep`) |
| `subfinder` / `amass` | — | **not covered** — pass a host list with `-T` |

The shims are a redirect for muscle memory, not a full port: prefer the native
form once you know it (it takes every vigolium flag). Recipes:
**`references/scanning.md`**.

## Reference router

Load the file that matches the task — don't read them all.

| Topic | Reference | Load when |
|-------|-----------|-----------|
| **Driving vigolium from an agent** | `references/agent-loop.md` | triage, `-j` contracts, replay, exports, exit codes |
| Scanning commands | `references/scanning.md` | scan / scan-url / scan-request / run flags, phases, strategies, output formats, replacing nuclei/ffuf/katana/httpx |
| Fuzzing | `references/fuzzing.md` | `vigolium fuzz` — positions, markers, attack modes, payload classes, anomaly scoring, matchers |
| Utility toolbox | `references/kit.md` | `vigolium kit` — secret-scan, js-beautify, oast, harvest, jwt-crack, wordlist, payload |
| Burp Suite / Caido | `references/burp.md` | Burp **or** Caido bridge setup, live history, Repeater/Organizer/Site map handoff, `--send-via-burp`, proxy channel — one loopback protocol, either vendor |
| AI agent modes | `references/agent-modes.md` | agent query / autopilot / swarm / audit / olium / triage / session, intensities, providers, templates |
| Auth & sessions | `references/auth.md` | `--auth-file` / `--auth`, YAML format, extract rules, authenticated scanning |
| Data & management | `references/data.md` | db, finding, traffic, module, extensions, js, config, scope, export, import, log, project, storage |
| Server mode | `references/server.md` | `vigolium server` — REST API, recording/MITM proxy, scan-on-receive, live mirror, endpoints |
| Ingesting traffic | `references/ingest.md` | `vigolium ingest` — local/remote, per-format input examples, spec flags |
| Writing extensions | `references/extensions.md` | custom JS scanner modules, `vigolium.*` API |
| Any specific flag | `references/flags.generated.md` | generated from the command tree — grep it by flag name |

> **Not covered?** `vigolium <command> -h` is the authoritative, version-matched
> flag list. Then search [docs.vigolium.com](https://docs.vigolium.com/) — start
> with the [cheat sheet](https://docs.vigolium.com/getting-started/cheat-sheet).
> This skill is a curated subset; the docs are the source of truth.

## Token discipline

The single most important habit when driving vigolium from an agent: **survey
cheap, drill narrow.**

```bash
# 1. Survey — metadata only, chosen fields.
vigolium finding -j --min-severity high --compact --fields id,severity,module_id,url

# 2. Drill — one finding, self-contained with its HTTP records.
vigolium finding -j --id 42 --with-records

# 0. Cheaper still — when the question is a shape, not a row set, count in SQL.
vigolium traffic -j --group-by status_code --host target.example
```

**Never dump rows to count them.** `--group-by` runs the same filters the listing
would and returns buckets, so "what status codes did we see" costs a dozen
integers instead of the corpus.

Under `--json`, bodies are preview-capped with `body_size`/`body_sha256`/
`body_truncated`, binaries are stubbed `body_omitted:"binary"`, and findings get
a ±240-char `response_evidence` snippet windowed on the match. The full
shaping-flag table (`--compact`, `--fields`, `--full-body`, `--with-records`,
`--min-severity`, `--pick`, `--markdown`, `--raw`, `--agentic-scan`) lives in
`references/agent-loop.md`.

Three things worth knowing about that habit — details in
`references/agent-loop.md`:

- **An unknown `--fields` name is a usage error** (exit 2) that lists the valid
  set for that view, so a typo fails loudly instead of returning a row with the
  key quietly missing. `--fields` composes with `--with-records`; use
  `--record-fields` / `--record-limit` to shape the nested records.
- **A gzip body over 1 MiB comes back as a prefix**, flagged `decoder_capped:
  true` with `body_size`/`body_sha256` describing the *whole* body. When you see
  that flag, do not conclude a string is absent — pull the body whole with
  `vigolium db export --format fs --uuid <uuid>`.
- **Read one record by identity with `traffic --uuid <uuid>`**, applied before
  pagination. A UUID as the positional term searches text, not identity.

## Global flags

These work on **every** command, and a per-command flag of the same name wins.
The ones an agent reaches for, beyond `-j`, `--db`, `--project-*`, `--format`,
`--no-color` and `--silent` covered above:

| Flag | Use it when |
|------|-------------|
| `--dump-traffic` | you need to see the actual bytes. Prints every request/response pair to **stderr**, Burp-style, bypassing the logger - so it composes with `-j` on stdout instead of corrupting it |
| `--debug` | a scan behaves unexpectedly; adds debug-level logging **including outgoing request lines**. `-v/--verbose` is the milder step |
| `--log-file <path>` | you want the run's logs as JSON on disk instead of scrolling past - the thing to attach to a bug report |
| `--mem-limit` | a big crawl is being OOM-killed. A soft `GOMEMLIMIT` ceiling; default auto is ⅓ of RAM, scaled down by `-P` so the children together stay under ⅔. Takes `off`, `6GiB`, or `50%`. An existing `GOMEMLIMIT` env var overrides it |
| `--read-only` | the database is evidence - no mkdir, no journal change, no checkpoint (see Invariants) |
| `--skip-dependency-check` | a container/CI run must not stall on the first-run chromium + nuclei-template check |
| `-M/--list-modules` | you want the module list without `module ls` |
| `--list-input-mode` | you forget which `-I` values exist; prints each with an example |
| `--width <n>` | table output is wrapping badly (default 70 columns) |
| `--config <path>` | pinning a config file explicitly, the way `--db` pins the store |

**`--dump-traffic` is the right debugging reflex, not `--debug`.** When the
question is "what did we actually send", it answers directly and keeps stdout
clean for a parser; `--debug` buries the same bytes in log lines.

## Invariants

Things `-h` won't tell you:

- **`-S` means `--stateless` everywhere**, and is accepted (as a no-op) on
  commands where it is meaningless — you never need a per-command acceptance
  table. The one exception is retiring: on `server`/`ingest` it is still a
  deprecated alias for `--scan-on-receive` and warns; **use the long
  `--scan-on-receive` there**.
- **Which DB a command opens:** `--db` → `$VIGOLIUM_DB_PATH` →
  `database.sqlite.path` in config → the built-in default. Pinning
  `$VIGOLIUM_DB_PATH` also makes **read** commands (`finding`/`traffic`/`log`/
  `fuzz -u`) treat that file as a stateless source — project scoping off — so an
  agent can export it once and every read/write lands in the same session DB.
  That flip only happens once the file **exists and is a usable vigolium store**;
  before the session's first scan the same reads stay project-scoped, so check
  `project_scoped` rather than assuming. It never turns a *scan* stateless (that
  would clash with `--db`). "Unusable" here means a path that can never *become* a
  database — a directory, or a parent dir that cannot be created or written — and
  that is a hard error rather than a silent fall-through to the shared default; a
  path that simply does not exist yet is fine and gets created. Every `-j`
  envelope reports the `db_path` it actually opened — assert it rather than
  trusting the pin. The envelope also reports `project_scoped`:
  `true` means a project filter was applied and `project_uuid` names it; `false`
  (under `-S`) means the rows span every project in the file.
- **Exit codes are a table, not a boolean.** `0` success · `1` error · `2` usage
  error (bad flag or combination) · `3` `fuzz --fail-on-match` matched · `4`
  `--fail-on <sev>` gate tripped. **`4` is not a failure** — the scan ran to
  completion and found something. `--soft-fail` forces `0` everywhere.
- **A positional argument to a scan command is always a target URL, never a
  file.** `vigolium scan https://a https://b` is three ways of saying the same
  thing as `-t`, and they merge with `-t`/`-T` with duplicates removed. But
  `vigolium scan targets.txt` does **not** read the file — it scans the literal
  string `targets.txt` as a target and reports `Targets: 1`. Files go through
  `-T` (target lists) or `-i` (specs and request exports).
- `--only` and `--skip` are mutually exclusive.
- **`--passive-only` sends no attack traffic** (passive modules only, secret
  detection included) and is on `scan`/`scan-url`/`scan-request` — **not** on
  `run`. Reach for it when the brief is "look, don't touch".
- `--format html`, `--format sqlite`, and any multi-value `--format` need a file
  destination: pass `-o/--output` **or** `--split-by-host` (which names per-host
  files from the hostname instead).
- `--format sqlite` additionally requires `-S` — it exports the standalone
  per-run DB. For the persisted DB use `vigolium export`.
- `--format fs` writes two sibling dirs (`<base>-traffic/`, `<base>-findings/`);
  `--split-by-host` is a no-op for it.
- `--stateless` on a **scan** is mutually exclusive with `--db` and with
  `--db-isolate`; `-o` is optional (without it results are simply discarded).
  On **read** commands (`finding`/`traffic`/`replay`) `-S` reads *from* `--db`.
- `--fail-on` writes output first, then sets the exit code. `--soft-fail`
  (global) overrides it. Under `-P` it is evaluated per child.
- `--split-by-host` only applies in stateless multi-target mode (`-S -T file`),
  and is required for `-P > 1`.
- `-m` and `--module-tag` **merge** (union); `--module-tag` is OR across tags.
- `--module-id` matches active **and** passive registries exactly; `-m` is a
  fuzzy match on active modules only.
- Server mode requires API-key auth unless `-A`/`--no-auth`.
- `db clean` with no selector is rejected. `db clean --all` needs `--force`.
- **Opening a database writes to it by default** — `mkdir -p`, journal PRAGMAs, a
  WAL checkpoint — so a plain read changes the file's SHA-256. `-S`/`--stateless`
  is a *scoping* mode, not a read-only one. Pass **`--read-only`** when the source
  is evidence: no directory creation, no journal change, no checkpoint, reads a
  `chmod 444` file, and errors on a missing path instead of creating one. Refused
  (exit 2) on commands that write.
- **`traffic --uuid` / `db ls --uuid`** select exact records before pagination.
  `replay -u` and `fuzz -u` also take a record UUID but **send traffic** — they
  are never what a read suggests.
- **The `query` hint in a `-j` envelope is always a valid read-only command**, so
  it is safe to run. Re-sending requires asking for `replay`/`fuzz` explicitly.
- **Errors under `-j` are JSON on stdout** (`{ok:false, error:{code,…}}`) with the
  human line on stderr. Branch on `error.code`; never `2>&1` into a parser.
- Every `-j` command uses the envelope, `--list-tables`/`--list-columns`
  and `log ls` included.
- `db export` validates its flags **before** opening `-o`, so a bad `--format`
  leaves the existing file intact.
- **`finding --id` takes the findings table's integer**, and rejects a UUID by
  naming the flag that *does* read that namespace (`traffic --uuid` for a stored
  record, `--agentic-scan` / `--scan-uuid` for a run). A UUID from another tool is
  not a vigolium identifier: find the finding by content (`--search`) and read the
  integer off that result. Never widen a failed exact lookup into a search that
  returns different findings.
- **`db export --uuid` narrows both halves of an `fs` tree** — the selected
  record *and the findings linked to it*, not every finding in the store.
- **`traffic --group-by <field>` counts instead of listing**, through the same
  filters (`host`, `method`, `status_code`, `response_content_type`, `source`,
  `scan_uuid`, `ip`, `is_authenticated`). Bounded by `--group-limit` (default 20,
  `0` = all); the tail is reported as `other_groups`/`other_records` rather than
  dropped. Reach for it before dumping rows to count them yourself.
- `replay` has no `--mutate` — payload fuzzing is `vigolium fuzz`.
- `fuzz` emits **no findings** and makes no verdict; it reports signals. Reach for
  `-a/--anomaly` before hand-writing matchers, then confirm hits with
  `scan-request -m <module>`.
- **Two wordlist knobs, not interchangeable.** `--discovery-wordlist <path>`
  seeds the *discovery* phase (`scan --discover`); `vigolium fuzz -w
  <builtin|path>` is the standalone fuzzing primitive with its own builtin lists
  (`dir-short`, `file-long`, …). The scan-phase knob was `--fuzz-wordlist`, which
  is still accepted as a deprecated alias — the names collided, they never meant
  the same thing. See `references/fuzzing.md` for the latter.
- **Pace flags apply whether or not you type them, and can be scoped to a
  phase.** `--rate-limit` is enforced at its documented default when unset (pass
  `--rate-limit 0` for genuinely no cap; a negative value is a usage error).
  All three dials take an optional phase qualifier, repeatable and mixable with
  the bare form: `--rate-limit known-issue-scan=20 --rate-limit 50`. Resolution
  is phase-scoped → global → strategy/config default, and `scan.started` reports
  what actually applied.
- **`--strategy lite` really is gentler now** — it carries a pace ceiling
  (concurrency 10 / rate 20 / max-per-host 10) on top of choosing fewer phases.
  The ceiling only ever *narrows*: an explicit `--concurrency` overrules it, and
  a config already gentler than `lite` is not dragged up to it.
- **`import -B` refuses an unfiltered pull.** Without `--host`/`--path`/
  `--method`/`--status`/`--search`/`--from`/`--to`/`-n`, it errors rather than
  copying the operator's entire proxy history — every host they have browsed,
  with those hosts' cookies — into your DB. `--all-hosts` opts in on purpose;
  `--yes` skips the pre-flight confirmation.
- **`traffic -B --save-to-vigolium-db` no longer truncates at 100.** An untyped
  `-n` is a *listing* default and does not bound the write; a typed `-n` is
  honored and says so when it bit.
- **`vigolium log` does not hang on a dead scan.** Auto-follow is off when stdout
  is a pipe or the scan's row is stale, and WAF notices above the `--tail` window
  are surfaced without needing `--full`. Pass `--follow` explicitly to force it.
- `traffic --replay` is a shortcut for `replay` in bulk mode; only `replay` can
  change the request (`-H`, `--auth-session`, `--target`, `--session-id`).
- On `replay`, `-H/--header` **overrides** a header and `--header-search`
  **filters** records; on `traffic`, `--header` is the filter.
- Agent commands need a configured provider (`agent.olium` in
  `vigolium-configs.yaml`); verify with `vigolium doctor --json`.
- Whitebox scanning is an agent feature — `--source <path|git-url|archive|gs://>`
  on `agent autopilot`/`swarm`/`audit`/`query`, not on `scan`.

## Strategies, phases & formats

**Strategy** picks which phases run: `lite` (assessment only), `balanced`
(default: + discovery/spidering/known-issue-scan), `deep` (+ external-harvest).
Select with `--strategy`; print the matrix with `vigolium strategy`.

**Intensity** is the one-flag shortcut on top: `--intensity quick|balanced|deep`
maps to a scanning profile **and** strategy at once (also honored by `agent
autopilot`/`swarm`). Explicit flags override it.

**Phases** (for `--only`/`--skip`, or `vigolium run <phase>`), canonical name +
aliases: `ingestion` (`ingest`,`ingesting`) · `probe`
(`probing`,`httpx`,`alive`,`sweep`) · `discovery`
(`discover`,`discovering`,`deparos`) · `external-harvest`
(`harvest`,`harvesting`,`external-harvester`) · `spidering`
(`spider`,`spitolas`,`crawl`,`crawling`,`crawler`) · `known-issue-scan`
(`cve`,`kis`,`known-issue`,`known-issues`) · `dynamic-assessment`
(`dast`,`audit`,`assessment`,`assess`) · `extension` (`ext`,`extensions`).

> `audit` is the one alias that is **not** universal: `--only audit` / `--skip
> audit` work, but `vigolium run audit` is rejected as ambiguous with `vigolium
> agent audit` (the AI source-code audit). Type `run dast` for the native phase.

**Input** (`-I`, OpenAPI/WSDL auto-detect): `urls` (default) · `openapi` ·
`swagger` · `wsdl` · `burp` · `curl` · `nuclei` · `har` · `postman` ·
`burpscope`. **Output** (`--format`, comma-combinable): `console` (default) ·
`jsonl` · `html` · `sarif` · `sqlite` · `fs`.

Full tables — strategy matrix, phase descriptions, per-format input examples,
spec flags, format constraints, precedence: **`references/scanning.md`**.
Whitebox/source-aware scanning is **not** a strategy — it is an agent feature
(`--source` on `agent audit`/`autopilot`/`swarm`/`query`).

## Utility toolbox (`vigolium kit`)

`vigolium kit` is a family of stateless one-shot primitives — no DB, no project
scope, no scan pipeline. Each reads `-`/stdin, takes `-j/--json`, and persists
nothing, so pipe the output. The seven commands (`secret-scan`, `js-beautify`,
`oast`, `harvest`, `jwt-crack`, `wordlist`, `payload`) are in the Command Router
above; usage, flags, and JSON shapes: **`references/kit.md`**.

## Recipes

Multi-step workflows only — single commands are in the router above.

### 1. Spec scan with auth

```bash
vigolium scan -I openapi -i spec.yaml -t https://api.target.example \
  --spec-header "Authorization: Bearer <token>" --fail-on high
```

### 2. CI gate with a shareable artifact

```bash
vigolium scan -t https://target.example \
  -S --format jsonl,html -o report.html --fail-on medium
```

Output is written before the gate fires, so the artifact exists even on failure.
`--soft-fail` forces exit 0 if the pipeline must not break.

### 3. Parallel fan-out across many targets

```bash
vigolium scan -T targets.txt -S --split-by-host -P 4 \
  --format sqlite -o run --fail-on high

# Merge the per-host databases back into one and query across all of them.
vigolium import --db combined.sqlite --glob-db 'run-*.sqlite'
vigolium finding --db combined.sqlite --min-severity high
```

`-P` requires `-S -T --split-by-host` (or `--db-isolate -T`). The gate is
evaluated per child; the batch fails only when every target fails.

### 4. Triage loop: survey → drill → confirm → report

```bash
vigolium finding -j --min-severity high --compact --fields id,severity,module_id,url
vigolium finding -j --id 42 --with-records
vigolium replay --finding-id 42 --pretty
vigolium finding --id 42 --markdown > finding-42.md
```

See `references/agent-loop.md` for the full contract.

### 5. Authenticated scan

```bash
# Inline session
vigolium scan -t https://target.example --auth "admin:Cookie:session=abc123"

# From an auth file (YAML/JSON, single session or a `sessions:` bundle)
vigolium scan -t https://target.example --auth-file ./auth.yaml

# Multi-session auth-diff testing (IDOR / BFLA)
vigolium scan -t https://target.example --auth-file ./sessions.yaml
```

Format, extract rules, and multi-step login flows: `references/auth.md`.

### 6. Whitebox: source + running app

```bash
# Autopilot runs an audit harness over the source, then scans the live target.
vigolium agent autopilot -t http://localhost:3000 --source ./src

# Source-only audit, no target required.
vigolium agent audit --source . --intensity balanced

# Only the changed code in a PR.
vigolium agent autopilot -t http://localhost:3000 --source ./src \
  --diff main...feature-branch
```

`--source` accepts a local path, git URL, `.zip`/`.tar.gz` archive, or `gs://`
object. Modes, intensities, and drivers: `references/agent-modes.md`.

### 7. Capture traffic from a proxy, then scan it

```bash
# Terminal 1 — capture, mirroring to a live file tree an agent can grep.
vigolium server --ingest-proxy-port 8080 --mirror-fs ./mirror

# Terminal 2 — scan what was captured.
vigolium scan --only dynamic-assessment -t https://target.example
```

Or auto-scan each request as it arrives: `vigolium server -t <url> --scan-on-receive`.

### 8. Custom detection logic in JavaScript

```bash
vigolium ext preset                                   # install examples
vigolium ext docs                                     # API reference
vigolium ext lint --ext ./custom-check.js             # validate
vigolium run extension -t https://target.example --ext ./custom-check.js
```

Writing modules against the `vigolium.*` API: `references/extensions.md`.

## Escape hatches

```bash
vigolium <command> -h          # authoritative flags for your installed version
vigolium --full-example        # worked examples grouped by section
vigolium doctor --json         # environment readiness (binaries, providers, templates)
vigolium module ls <keyword>   # find a module by topic
vigolium skills get --full     # print this skill and every reference
```

## Resources

- **Website**: [www.vigolium.com](https://www.vigolium.com/)
- **Documentation**: [docs.vigolium.com](https://docs.vigolium.com/)
- **GitHub**: [github.com/vigolium/vigolium](https://github.com/vigolium/vigolium)
