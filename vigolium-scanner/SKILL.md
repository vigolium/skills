---
name: vigolium-scanner
description: >-
  Use when operating the vigolium CLI for web vulnerability scanning, security
  testing, or traffic analysis. Covers scanning a URL/spec/raw request, running
  AI agent scans (autopilot, swarm, audit, query), triaging findings, confirming
  them with replay/fuzz, handing off to Burp, browsing stored traffic, writing
  JavaScript scanner extensions, and managing projects, exports, and config.
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
- **Two JSON contracts — don't confuse them:**
  - `-j/--json` on `finding`/`traffic`/`db` → **one** compact, token-bounded
    object. Parse this during triage.
  - `--format jsonl` / `export` → bulk `{"type":…,"data":{…}}` stream, one per
    line, full fidelity. Archival, not triage.
- **Non-interactive by default.** TUI is opt-in (`--tui`). Destructive commands
  need `--force`. Use `--no-color` (or `NO_COLOR=1`) for clean text.
- **Everything is project-scoped** — `--project-name`, `--project-uuid`, or
  `VIGOLIUM_PROJECT`.
- **JSON summaries hand you the next command.** Agentic scans and `fuzz` emit a
  `query` field with a ready follow-up. Run that rather than composing your own.

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

## Command router

| I need to… | Use |
|---|---|
| Scan one or more target URLs | `vigolium scan -t <url>` |
| Scan a single URL with custom method/headers | `vigolium scan-url <url> --method POST --body '...'` |
| Scan a raw HTTP request from file/stdin | `vigolium scan-request -i request.txt` |
| Run only one scan phase | `vigolium run <phase>` or `scan --only <phase>` |
| Tune scan aggressiveness (phases + profile) | `vigolium scan -t <url> --intensity quick\|balanced\|deep` |
| Content discovery with a custom wordlist | `vigolium scan -t <url> --discover --fuzz-wordlist ./words.txt` |
| Import an OpenAPI/Swagger spec and scan | `vigolium scan -I openapi -i spec.yaml -t <base-url>` |
| Import Burp/HAR/cURL traffic | `vigolium scan -I burp -i export.xml` |
| Filter modules by tag | `vigolium scan -t <url> --module-tag spring --module-tag injection` |
| Ingest traffic without scanning | `vigolium ingest -t <url> -I openapi -i spec.yaml` |
| Start the API server | `vigolium server` |
| Start server and auto-scan new traffic | `vigolium server -t <url> -S` |
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
| Browse findings | `vigolium finding` or `vigolium db ls findings` |
| Compact agent JSON + linked records | `vigolium finding -j --with-records --min-severity high` |
| Re-send one request + baseline diff | `vigolium replay --record-uuid <uuid>` |
| Re-confirm a finding's evidence | `vigolium replay --finding-id 42` |
| Bulk re-send stored traffic by pattern | `vigolium replay <search> --proxy http://127.0.0.1:8080` |
| Re-send under a different session | `vigolium replay -u <uuid> -H 'Cookie: session=other'` |
| Payload / insertion-point fuzzing | `vigolium fuzz -u <uuid> --point URL_PARAM:id --class sqli -a` |
| Fuzz a wordlist at a `FUZZ` marker | `vigolium fuzz https://t/FUZZ -w file-long --match-status-code 200,301` |
| Hand a finding to Burp | `vigolium finding --id 42 --push-to-burp -B http://127.0.0.1:9009` |
| Pull live Burp Proxy history into the DB | `vigolium import -B http://127.0.0.1:9009` |
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
| Health check | `vigolium doctor` |

## Reference router

Load the file that matches the task — don't read them all.

| Topic | Reference | Load when |
|-------|-----------|-----------|
| **Driving vigolium from an agent** | `references/agent-loop.md` | triage, `-j` contracts, replay, exports, exit codes |
| Scanning commands | `references/scanning.md` | scan / scan-url / scan-request / run flags, phases, strategies, output formats |
| Fuzzing | `references/fuzzing.md` | `vigolium fuzz` — positions, markers, attack modes, payload classes, anomaly scoring, matchers |
| Burp Suite | `references/burp.md` | bridge setup, live history, Repeater/Organizer/Site map handoff, `--send-via-burp`, proxy channel |
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
```

| Flag | Effect |
|------|--------|
| `--compact` | metadata only, drop bodies |
| `--fields a,b,c` | project to just these top-level keys |
| `--full-body` | complete decoded bodies (exploit writing) |
| `--with-records` | (finding) embed linked HTTP records |
| `--min-severity` | (finding) threshold expands upward |
| `--pick N` | (finding) keep the Nth result — `2`, `1,3`, `2-4` |
| `--markdown` | (finding/traffic) render as Markdown instead of JSON |

Under `--json`, bodies are preview-capped with `body_size`/`body_sha256`/
`body_truncated`, binaries are stubbed `body_omitted:"binary"`, and findings get
a ±240-char `response_evidence` snippet windowed on the match.

## Invariants

Things `-h` won't tell you:

- **`-S` is two different flags.** `--stateless` on `scan`/`export`/`finding`/
  `replay`/`agent audit`; `--scan-on-receive` on `server`/`ingest`.
- **Which DB a command opens:** `--db` → `$VIGOLIUM_DB_PATH` →
  `database.sqlite.path` in config → the built-in default. Pinning
  `$VIGOLIUM_DB_PATH` also makes **read** commands (`finding`/`traffic`/`fuzz -u`)
  treat that file as a stateless source — project scoping off — so an agent can
  export it once and every read/write lands in the same session DB. It never
  turns a *scan* stateless (that would clash with `--db`).
- `--only` and `--skip` are mutually exclusive.
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
- `replay` has no `--mutate` — payload fuzzing is `vigolium fuzz`.
- `fuzz` emits **no findings** and makes no verdict; it reports signals. Reach for
  `-a/--anomaly` before hand-writing matchers, then confirm hits with
  `scan-request -m <module>`.
- **Two wordlist knobs, not interchangeable.** `--fuzz-wordlist <path>` seeds the
  *discovery* phase (`scan --discover`); `vigolium fuzz -w <builtin|path>` is the
  standalone fuzzing primitive with its own builtin lists (`dir-short`,
  `file-long`, …). See `references/fuzzing.md` for the latter.
- `traffic --replay` is a shortcut for `replay` in bulk mode; only `replay` can
  change the request (`-H`, `--auth-session`, `--target`, `--session-id`).
- On `replay`, `-H/--header` **overrides** a header and `--header-search`
  **filters** records; on `traffic`, `--header` is the filter.
- Agent commands need a configured provider (`agent.olium` in
  `vigolium-configs.yaml`); verify with `vigolium doctor --json`.
- Whitebox scanning is an agent feature — `--source <path|git-url|archive|gs://>`
  on `agent autopilot`/`swarm`/`audit`/`query`, not on `scan`.

## Scanning strategies

Strategies control which phases run. Use `--strategy <name>`.

| Strategy | ExtHarvest | Discovery | Spidering | KnownIssueScan | Assessment | Source-Aware |
|----------|:---------:|:---------:|:---------:|:--------------:|:----------:|:------------:|
| `lite` | no | no | no | no | yes | no |
| `balanced` *(default)* | no | yes | yes | yes | yes | no |
| `deep` | yes | yes | yes | yes | yes | no |
| `whitebox` | no | yes | no | yes | yes | yes |

Default lives in `scanning_strategy.default_strategy`. Print the table with
`vigolium strategy` (no `ls` subcommand).

**Intensity** is the one-flag shortcut on top of strategies:
`--intensity quick|balanced|deep` maps to a scanning profile **and** strategy in a
single flag (also honored by `agent autopilot`/`swarm`). Explicit flags always
override it — `--intensity deep --scanning-profile foo` keeps `deep`'s strategy
but your profile. Full precedence: `references/scanning.md`.

## Scan phases

Use `--only <phase>` to isolate one or `--skip <phase>` to drop some.

| Phase | Aliases | Description |
|-------|---------|-------------|
| `ingestion` | — | Parse and store input into the database |
| `discovery` | `deparos`, `discover` | Adaptive content discovery |
| `external-harvest` | — | Wayback / Common Crawl / OTX URL aggregation |
| `spidering` | `spitolas` | Headless-browser crawling for JS-driven routes |
| `known-issue-scan` | `cve`, `kis`, `known-issues` | Nuclei templates + Kingfisher secrets |
| `dynamic-assessment` | `audit`, `dast`, `assessment` | Core active + passive vulnerability scanning |
| `extension` | `ext` | JavaScript extension modules only |

Run one directly: `vigolium run discover -t <url>`.

## Input formats

`-I <format>` selects the input type; OpenAPI and WSDL auto-detect from content.

| Format | Flag | Example |
|--------|------|---------|
| URLs *(default)* | `-I urls` | `-t https://target.example` or `-T targets.txt` |
| OpenAPI 3.x | `-I openapi` | `-I openapi -i spec.yaml -t https://api.target.example` |
| Swagger 2.0 | `-I swagger` | `-I swagger -i swagger.json` |
| WSDL / SOAP | `-I wsdl` | `-I wsdl -i service.wsdl -t https://soap.target.example` — one SOAP POST per operation; a `.svc`/`.asmx` URL auto-fetches its WSDL |
| Burp XML | `-I burp` | `-I burp -i burp-export.xml` |
| cURL commands | `-I curl` | `-I curl -i requests.txt` |
| Nuclei templates | `-I nuclei` | `-I nuclei -i templates/` |
| HAR archive | `-I har` | `-I har -i traffic.har` |
| Postman collection | `-I postman` | `-I postman -i collection.json` |
| Burp scope export | `-I burpscope` | `-I burpscope -i burp-scope.json` — expands a program's scope into seed URLs (content-sniffed on `-T` too) |
| stdin | — | `cat urls.txt \| vigolium scan -i -` |

OpenAPI extras: `--spec-url` (use servers from the spec), `--spec-header`
(auth), `--spec-var` (parameter values), `--spec-default` (fallback).
WSDL reuses `--spec-header` (auth) and `--spec-var` (override a body element by
its local name); `-t` overrides only the endpoint host, keeping the WSDL path.

## Output formats

| Format | Flag | Notes |
|--------|------|-------|
| Console *(default)* | `--format console` | human-readable tables to stderr |
| JSONL | `--format jsonl` | bulk `{"type":…,"data":{…}}` stream |
| HTML | `--format html -o report.html` | interactive ag-grid report; requires `-o` |
| SQLite | `--format sqlite -S -o run.sqlite` | standalone per-run DB via `VACUUM INTO`; requires `-S` + `-o`; aliases `sqlite3`, `db` |
| Filesystem tree | `--format fs -o run` | browsable `run-traffic/` + `run-findings/`; see `references/agent-loop.md` |

Combine with commas: `--format jsonl,html -o report.html`.

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

Or auto-scan each request as it arrives: `vigolium server -t <url> -S`.

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
