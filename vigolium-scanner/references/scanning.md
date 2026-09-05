# Scanning Commands Reference

> **Related:** [agent-loop.md](agent-loop.md) for triage · [flags.generated.md](flags.generated.md) for any flag · `vigolium scan -h`

Complete flag reference for `scan`, `scan-url`, `scan-request`, and `run` commands.

## Choosing a scan command

- **`scan -t <url>`** — broad. Give it a bare host or root URL and it expands the
  surface itself (discovery, spidering, the full phase pipeline per `--strategy`).
- **`scan-url` / `scan-request`** — precision. Use these only when you're testing
  **one specific request**: it already has full query params, a deep multi-segment
  path, or custom headers/method/body that must be sent verbatim. They scan that
  single request (phases are opt-in via `--discover`/`--spider`).

A full `scan` can exceed 30 min per target — discovery defaults to
`--discover-max-time 1h`, spidering to `--spider-max-time 30m`, and
`--scanning-max-duration` defaults to `0s` (**no total cap**). **Run a full scan
in the background**, or bound it explicitly:

```bash
vigolium scan -t https://target.example --scanning-max-duration 30m
```

To iterate on one part of the pipeline instead of the whole thing, run a phase
directly with `vigolium run <phase>` (see [run](#run)) — e.g. `run spidering` or
`run discovery`.

## Table of Contents

- [scan](#scan)
- [scan-url](#scan-url)
- [scan-request](#scan-request)
- [run](#run)
- [Strategy and Phase Interaction](#strategy-and-phase-interaction)
- [Replacing standalone recon tools](#replacing-standalone-recon-tools)

---

## scan

**Usage:** `vigolium scan [flags]`

Run a full vulnerability scan pipeline. Supports multiple targets, input formats, phase control, and strategy presets.

### Output flags (scan & run)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--output` | `-o` | string | — | Write findings to specified output file |
| `--stats` | — | bool | `false` | Show live progress stats during scanning |
| `--include-response` | — | bool | `false` | Include full HTTP response body in output |
| `--omit-response` | — | bool | `false` | Omit raw request/response bytes (smaller files; drops the `.resp.*` files under `--format fs`) |
| `--fail-on` | — | string | — | Exit non-zero when a finding at/above this severity is present (`info`,`suspect`,`low`,`medium`,`high`,`critical`); output written first, `--soft-fail` overrides |
| `--split-by-host` | — | bool | `false` | Stateless multi-target (`-S -T file`): write per-host output files (`base-<host>.<ext>`); required for `-P > 1` fan-out; no-op for `--format fs` |
| `--stateless` | — | bool | `false` | Use a temporary database, export results to `--output`, then discard |
| `--upload-results` | — | bool | `false` | Upload scan results to cloud storage after completion (requires storage config) |

Stateless mode is great for ephemeral CI/CD runs — it creates a temp SQLite file, runs the full scan against it, writes the export/report to `--output`, then deletes the DB (including WAL/SHM sidecars). Requires `--output`; mutually exclusive with `--db`. Combine with `--format jsonl`, `--format html`, `--format fs`, or `--format sqlite` for shareable artifacts (`sqlite` requires `-S`).

`--format` accepts `console` (default), `jsonl`, `html`, `sqlite`, and `fs` (comma-separated for multiple):
- **`fs`** — a flat, browsable tree (`<base>-traffic/` + `<base>-findings/`) with per-host `.req` / `.resp.headers` / `.resp.body` / `.md` files and a jq-friendly `index.json`. No `-o` → `vigolium-traffic/` + `vigolium-findings/`. Works with or without `-S`. `--omit-response` drops the `.resp.*` files.
- **`sqlite`** (aliases `sqlite3`, `db`) — dumps the standalone per-run DB to `<output>.sqlite` via `VACUUM INTO`. Requires `-S/--stateless` + `-o`. Reopen with `vigolium finding/traffic -S --db <file>.sqlite`.

### Request flags (scan & run)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--header` | `-H` | []string | — | Add custom HTTP header (repeatable, e.g. -H 'Auth: Bearer token') |
| `--advanced-options` | `-a` | map | — | Module-specific options as key=value (e.g. -a xss.dom=true) |
| `--retries` | — | int | `1` | Retry attempts for failed requests |
| `--stream` | — | bool | `false` | Process targets as a stream without buffering or deduplication |

### Input Format flags (scan & run)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--required-only` | bool | `false` | Parse only required fields from input format (ignore optional) |
| `--skip-format-validation` | bool | `false` | Skip validation of input file format |

### Other flags (scan & run)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--auth-file` | []string | — | Path to auth file (YAML/JSON, single session or `sessions:` bundle), or bare name resolved against session_dir. Repeatable. |
| `--auth` | []string | — | Inline session in `name:Header:value` format. Repeatable. |
| `--oast-url` | string | — | Fixed out-of-band callback URL (overrides auto-generated interactsh URL) |

### Parallel, DB & module flags (scan & run)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--parallel` | `-P` | int | `1` | Scan up to N targets concurrently as isolated child processes (requires `-S -T --split-by-host`, OR `--db-isolate -T`) |
| `--db-isolate` | — | bool | `false` | Scan into a private temp DB, then merge into `--db` at the end (SQLite only, not with `--stateless`) |
| `--resume` | — | bool | `false` | Resume a prior `-S -T --split-by-host -P` run from its `<output>.progress.json` manifest |
| `--follow-subdomains` | — | bool | `false` | Pull in-scope subdomains found in responses into the scan (auto-on at `--intensity deep`) |
| `--module-id` | — | []string | — | Run exactly these module IDs (exact match against **both** active + passive registries; unlike `-m`, also selects passive) — **scan/scan-url/scan-request only, not `run`** |
| `--passive-only` | — | bool | `false` | Run only passive modules (no active scan traffic) — **scan/scan-url/scan-request only, not `run`** |
| `--print-finding` / `--print-traffic` / `--print-traffic-tree` | — | bool | `false` | After the scan, print findings / raw traffic / traffic tree to stdout (pairs with `-S`/`--silent`) |
| `--report-url` | — | string | — | URL for the "Raw Report URL" button in HTML reports |
| `--no-waf-pacing` | — | bool | `false` | Disable proactive CDN/WAF-edge pacing |
| `--no-tech-filter` | — | bool | `false` | Disable tech-stack fingerprint gating of modules |

### Content Discovery flags (scan & run)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--discover` | bool | `false` | Enable content discovery phase before scanning |
| `--discover-max-time` | duration | `1h` | Max time for content discovery per target |
| `--discovery-wordlist` | string | — | Custom wordlist seeding the discovery phase (enables fuzzing on the fly). Formerly `--fuzz-wordlist`, still accepted as a deprecated alias. **Not** the same knob as `vigolium fuzz -w`. |
| `--no-prefix-breaker` | bool | `false` | Disable per-prefix circuit breaker that stops trap-directory recursion |
| `--port-sweep-ports` | string | — | Override the alternate HTTP(S) ports swept on CLI target hosts (comma-separated; the sweep runs at `--intensity deep` or with `--follow-subdomains`) |

### Browser Spidering flags (scan & run)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--spider` | — | bool | `false` | Enable browser-based spidering phase before scanning |
| `--spider-max-time` | — | duration | `30m` | Max time for spidering per target |
| `--browser-engine` | `-E` | string | `chromium` | Browser engine: chromium, ungoogled, fingerprint |
| `--browsers` | `-b` | int | `1` | Number of parallel browser instances for spidering |
| `--headless` | — | bool | `true` | Run browser in headless mode |
| `--no-cdp` | — | bool | `false` | Disable Chrome DevTools Protocol event listener detection |
| `--no-forms` | — | bool | `false` | Disable automatic form detection and filling during spidering |
| `--headed` | — | bool | `false` | Show the browser window during spidering (sugar for `--headless=false`; wins when both are set) |
| `--no-carry-browser-session` | — | bool | `false` | Do not carry the spidering browser's cleared session (cookies + UA) into discovery/scanning (on by default when `--spider` runs; scoped to the same host, respects `-H`) |

### External Harvest flags (scan & run)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--external-harvest` | bool | `false` | Enable external intelligence gathering phase (Wayback, CT logs, etc.) |

### KnownIssueScan flags (scan & run)

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--known-issue-scan-tags` | []string | — | Nuclei template tags to include |
| `--known-issue-scan-exclude-tags` | []string | — | Nuclei template tags to exclude |
| `--known-issue-scan-severities` | []string | — | Filter Nuclei templates by severity (critical,high,medium,low,info) |
| `--known-issue-scan-templates-dir` | string | — | Custom Nuclei templates directory |

> **Source-aware / SAST scanning is an agent feature**, not a native `scan`/`run` phase.
> Use `vigolium agent audit --source <path-or-git-url>` (security code audit) or
> `vigolium agent query --source <path> -t code-review`. See `references/agent-modes.md`.

### Examples

```bash
# Basic scan
vigolium scan -t https://example.com

# Multiple targets
vigolium scan -t https://example.com -t https://api.example.com

# Targets from file
vigolium scan -T targets.txt

# Deep strategy with discovery
vigolium scan -t https://example.com --strategy deep

# Phase isolation
vigolium scan -t https://example.com --only dynamic-assessment
vigolium scan -t https://example.com --only ext --ext ./custom-check.js
vigolium scan -t https://example.com --skip discovery,spidering

# Specific modules
vigolium scan -t https://example.com -m xss-reflected,sqli-error

# Custom scanning profile
vigolium scan -t https://example.com --scanning-profile aggressive

# JSONL output
vigolium scan -t https://example.com --format jsonl -o results.jsonl

# HTML report
vigolium scan -t https://example.com --format html -o report.html

# Filesystem tree (run-traffic/ + run-findings/) — browsable with ls/grep/jq
vigolium scan -t https://example.com --format fs -o run

# Standalone per-run SQLite DB (requires -S)
vigolium scan -t https://example.com -S --format sqlite -o run.sqlite

# Fail the pipeline on any high/critical finding
vigolium scan -t https://example.com --fail-on high

# With proxy
vigolium scan -t https://example.com --proxy http://127.0.0.1:8080

# Speed tuning (bare form applies to the whole invocation)
vigolium scan -t https://example.com -c 100 --rate-limit 200

# Per-phase speed: cap the nuclei phase only
vigolium scan -t https://example.com --rate-limit known-issue-scan=20

# Watch it run from a program (human console stays on stderr)
vigolium scan -t https://example.com --events ndjson 2>/dev/null | jq -c .

# Source-aware / whitebox scanning is an agent feature (see agent-modes.md)
vigolium agent autopilot -t https://example.com --source ./src

# Source-aware via git clone (--source accepts a git URL)
vigolium agent swarm -t https://example.com --source https://github.com/org/repo

# OpenAPI scan
vigolium scan -I openapi -i openapi.yaml -t https://api.example.com

# WSDL / SOAP scan (one SOAP POST per operation; a .svc/.asmx URL auto-fetches the WSDL)
vigolium scan -I wsdl -i service.wsdl -t https://api.example.com
vigolium scan -i https://api.example.com/Service.svc

# Burp import scan
vigolium scan -I burp -i burp-export.xml -t https://example.com

# Pipe from stdin
cat urls.txt | vigolium scan -i -

# Filter modules by tag
vigolium scan -t https://example.com --module-tag spring --module-tag injection

# Run extension during scan
vigolium scan -t https://example.com --ext custom-check.js

# Extensions-only scan
vigolium scan -t https://example.com --only extension --ext custom-check.js
```

---

## scan-url

**Usage:** `vigolium scan-url <url> [flags]`

Scan a single URL for vulnerabilities. Designed for quick, targeted scans and AI agent integration. Returns JSON output with findings.

> Precision command — for one specific request; for broad surface expansion use `scan -t` (see [Choosing a scan command](#choosing-a-scan-command)).

### scan-url specific flags

**Spidering:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--spider` | bool | `false` | Run browser-based spidering before scanning |

**Discovery:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--discover` | bool | `false` | Run content discovery before scanning |

**Harvest:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--external-harvest` | bool | `false` | Run external intelligence harvesting before scanning |

**Request:**

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--header` | `-H` | []string | — | Custom header (repeatable) |

**Other:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--method` | string | `GET` | HTTP method |
| `--body` | string | — | Request body |
| `--known-issue-scan` | bool | `false` | Run known issue scan (Nuclei/Kingfisher) |
| `--no-passive` | bool | `false` | Skip passive modules |

### Examples

```bash
# Simple GET scan
vigolium scan-url https://example.com/api/users

# POST with body
vigolium scan-url https://example.com/login \
  --method POST --body '{"user":"admin","pass":"test"}' \
  -H "Content-Type: application/json"

# With discovery phase
vigolium scan-url https://example.com --discover

# Specific modules, no passive
vigolium scan-url https://example.com/api -m xss-reflected --no-passive
```

---

## scan-request

**Usage:** `vigolium scan-request [flags]`

Read a raw HTTP request from file or stdin and run scanner modules against it. Designed for pipeline integration and AI agent workflows.

> Precision command — for one specific request; for broad surface expansion use `scan -t` (see [Choosing a scan command](#choosing-a-scan-command)).

### scan-request specific flags

**Spidering:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--spider` | bool | `false` | Run browser-based spidering before scanning |

**Discovery:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--discover` | bool | `false` | Run content discovery before scanning |

**Harvest:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--external-harvest` | bool | `false` | Run external intelligence harvesting before scanning |

**Other:**

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--input` | `-i` | string | `-` (stdin) | Input file or stdin |
| `--target` | `-t` | string | — | Override target URL (scheme://host) |
| `--known-issue-scan` | — | bool | `false` | Run known issue scan |
| `--no-passive` | — | bool | `false` | Skip passive modules |

### Examples

```bash
# From file
vigolium scan-request -i raw-request.txt

# From stdin
echo -e "GET /api/users HTTP/1.1\r\nHost: example.com\r\n" | vigolium scan-request

# With target override
vigolium scan-request -i request.txt --target https://staging.example.com

# With discovery
vigolium scan-request -i request.txt --discover
```

---

## run

**Usage:** `vigolium run <phase> [flags]`

**Aliases:** `r`

Run a single scan phase directly. Equivalent to `vigolium scan --only <phase>`.

### Valid phases

| Phase | Aliases | Description |
|-------|---------|-------------|
| `ingestion` | — | Parse and store input into the database |
| `discovery` | `deparos`, `discover` | Adaptive content discovery |
| `external-harvest` | — | Wayback / Common Crawl / OTX URL aggregation |
| `spidering` | `spitolas` | Headless-browser crawling for JS-driven routes |
| `known-issue-scan` | `cve`, `kis`, `known-issues` | Nuclei templates + Kingfisher secrets |
| `dynamic-assessment` | `dast`, `assessment` | Core active + passive vulnerability scanning |
| `extension` | `ext` | JavaScript extension modules only |

`audit` is also an alias for `dynamic-assessment` on `--only` / `--skip`, but **not** as a `vigolium run` argument: `vigolium run audit` is rejected because it reads as the AI source-code audit (`vigolium agent audit`). Use `vigolium run dast` for the native module scan.

The `run` command accepts the same flag groups as `scan` (Spidering, Discovery, Harvest, KnownIssueScan, Input Format, Request, Output, and `--oast-url`), **except** the module-selection flags `--module-id` / `--passive-only`, which are only on `scan` / `scan-url` / `scan-request`.

### Examples

```bash
vigolium run discover -t https://example.com
vigolium run spidering -t https://example.com
vigolium run dast -t https://example.com
vigolium run dast -t https://example.com --module-tag spring
vigolium run external-harvest -t https://example.com
vigolium run known-issue-scan -t https://example.com
vigolium run known-issue-scan -t https://example.com --known-issue-scan-tags cve --known-issue-scan-severities critical,high
vigolium run extension -t https://example.com --ext custom-check.js
vigolium run ext -t https://example.com --ext ./my-scanner.js
vigolium run deparos -t https://example.com
vigolium run dynamic-assessment -t https://example.com
```

---

## Strategy and Phase Interaction

### Strategy matrix

Strategies control which phases run. Select with `--strategy <name>`; print the
table with `vigolium strategy` (no `ls` subcommand). Default lives in
`scanning_strategy.default_strategy`.

| Strategy | ExtHarvest | Discovery | Spidering | KnownIssueScan | Assessment |
|----------|:---------:|:---------:|:---------:|:--------------:|:----------:|
| `lite` | no | no | no | no | yes |
| `balanced` *(default)* | no | yes | yes | yes | yes |
| `deep` | yes | yes | yes | yes | yes |

Source-aware / whitebox scanning is **not** a strategy — it is an agent feature
(`agent audit`/`autopilot`/`swarm` `--source`; see the SAST note below).

### Precedence

1. `--only <phase>` overrides everything — only that phase runs, heuristics disabled
2. `--skip <phase>` disables specific phases while keeping all others
3. `--strategy <name>` sets baseline phase configuration
4. Individual phase flags (`--discover`, `--spider`, etc.) override strategy settings
5. Config file `scanning_strategy.default_strategy` provides the lowest-precedence default

### Heuristics

- Default: `--heuristics-check basic`
- Levels: `none`, `basic`, `advanced`
- `basic` probes target root pages to detect content type (HTML / JSON / blank) and skips spidering for non-HTML targets
- `advanced` adds deep HTML analysis to detect SPA frameworks and optimize phase selection
- `none` runs all enabled phases unconditionally
- `--skip-heuristics` is shorthand for `--heuristics-check=none`
- `--only` automatically disables heuristics
- Precedence: `--skip-heuristics` > `--heuristics-check` > config > `basic`

### Intensity Presets

`--intensity quick|balanced|deep` is a cross-cutting preset that maps to a scanning profile + strategy. It is also honored by `agent autopilot` and `agent swarm` with backend-specific defaults. Explicit flags always override the preset — e.g. `--intensity deep --scanning-profile foo` applies `deep`'s strategy but your custom profile.

### Scanning Pace

Speed settings have a layered precedence:

1. A **phase-qualified** CLI flag (`--rate-limit known-issue-scan=20`) — highest
2. A bare CLI flag (`-c`, `--rate-limit`, `--max-per-host`)
3. `--scanning-max-duration` — overrides `scanning_pace.max_duration`
4. Config `scanning_pace` section — per-phase max_duration and duration_factor
5. The strategy's own pace ceiling (`lite` only — see below)
6. Built-in defaults — lowest

**The pace flags apply whether or not you type them.** `--rate-limit` used to be
enforced only when passed, so silence meant *unlimited* while the help text
advertised 100. It now applies at its documented default; pass `--rate-limit 0`
for genuinely no cap. A negative value is a usage error (exit 2), not
"unlimited".

**Each of the three dials takes an optional phase qualifier, repeatable and
mixable with the bare form:**

```bash
# Cap nuclei without capping the ~200 native modules, which pace themselves.
vigolium scan -t https://target.example \
  --only known-issue-scan,dynamic-assessment \
  --rate-limit known-issue-scan=20

# Bare form, unchanged.
vigolium scan -t https://target.example --rate-limit 50

# Mixed: 50 rps everywhere, 20 for the nuclei phase.
vigolium scan -t https://target.example --rate-limit 50 --rate-limit kis=20
```

Qualifiers accept the same phase aliases as `--only`/`--skip` (`kis`, `cve`,
`deparos`, `dast`, …). A qualifier naming a phase this run does not execute
warns rather than failing. Resolution per phase is phase-scoped → global →
strategy/config default, and `scan.started` (under `--events`) reports the
resolved `pace` plus a `phase_pace` table for any phase that differs — assert
what applied instead of inferring it from the flags you passed.

**`--strategy lite` carries a pace ceiling** (concurrency 10 / rate 20 /
max-per-host 10) on top of selecting fewer phases. Previously `lite` and
`balanced` opened a crawl identically, so a name every operator reads as
"gentler on the target" was gentler in module count alone. The ceiling only ever
*narrows*: an explicit `--concurrency` overrules it, and a config already gentler
than `lite` is not dragged up to it. `balanced` is the baseline and `deep` ships
no ceiling.

**`known-issue-scan` opens conservatively when unattended.** It runs nuclei,
which owns its own HTTP stack and does not pace itself, so its no-flags defaults
are 20 rps / 10 host-concurrency. Before its phase starts, vigolium probes each
target host through the shared requester: a host whose edge is already filtering
is **dropped from the target list** (and reported as a `waf.block` event) rather
than scanned, and the per-host limiter's verdict narrows nuclei's own knobs. A
sentinel re-probes during the run and curtails the phase if a majority of hosts
start filtering.

### CI Output

- `--ci-output-format` enables CI-friendly output: JSONL findings only, no color, no banners
- Equivalent to combining `--format jsonl --silent`
- Useful for CI/CD pipelines that parse JSON output

### Valid `--only` Phases

The following phases can be used with `--only` and `--skip`:

`ingestion`, `discovery`, `external-harvest`, `known-issue-scan`, `spidering`, `dynamic-assessment` (aliases `audit`, `dast`, `assessment`), `extension`

### HTML Format Constraints

- `--format html` requires `-o/--output`
- In `scan` mode with `--only`, HTML is only supported for `discovery` and `spidering` phases
- The `export` command supports HTML for all data

### Filesystem & SQLite Format Constraints

- `--format fs` writes two sibling dirs off the `-o` base (`<base>-traffic/` + `<base>-findings/`); with no `-o` it defaults to `vigolium-traffic/` + `vigolium-findings/` in the cwd. Available on `scan` / `scan-url` / `scan-request` / `run`, `export`, and `db export`. `--split-by-host` is a no-op (fs already splits per host). For `scan-url` / `scan-request`, pass `-o`, `-S`, or a phase flag so the request routes through the runner that writes the tree
- `--format sqlite` requires `-S/--stateless` **and** `-o/--output`; aliases `sqlite3` / `db`. Under `--split-by-host` each file is `<base>-<host>.sqlite`. Reopen with `vigolium finding/traffic -S --db <file>.sqlite`

---

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

**`-i` vs `-T`:** `-i` reads a *spec / export* and expands it into requests; `-T`
reads a *list of target URLs*, one per line. Pointing `-T` at a YAML spec makes
every line of that file a target. The only genuine target-list formats are
`urls` and `burpscope`.

OpenAPI extras: `--spec-url` (use servers from the spec), `--spec-header`
(auth), `--spec-var` (parameter values), `--spec-default` (fallback). WSDL
reuses `--spec-header` (auth) and `--spec-var` (override a body element by its
local name); `-t` overrides only the endpoint host, keeping the WSDL path.

## Output formats

| Format | Flag | Notes |
|--------|------|-------|
| Console *(default)* | `--format console` | human-readable tables to stderr |
| JSONL | `--format jsonl` | bulk `{"type":…,"data":{…}}` stream |
| HTML | `--format html -o report.html` | interactive ag-grid report; requires `-o` |
| SARIF | `--format sarif -o out.sarif` | SARIF 2.1.0 for GitHub code scanning / DefectDojo |
| SQLite | `--format sqlite -S -o run.sqlite` | standalone per-run DB via `VACUUM INTO`; requires `-S` + `-o`; aliases `sqlite3`, `db` |
| Filesystem tree | `--format fs -o run` | browsable `run-traffic/` + `run-findings/`; see [agent-loop.md](agent-loop.md) |

Combine with commas: `--format jsonl,html -o report.html`.

### Exit-Code Gating

- `--fail-on <sev>` (`scan` / `run` / `scan-url` / `scan-request`) makes the command exit non-zero when a finding at/above `<sev>` was produced. Accepted (ascending): `info`, `suspect`, `low`, `medium`, `high`, `critical`
- Output is always written first; the gate fires afterward
- `--soft-fail` (global) forces exit 0 even when the gate (or any other error) trips
- Under `-P` / `--split-by-host` the gate is evaluated per child; the parent batch exits non-zero only when every target fails

### Source-Aware / SAST

Static analysis and source-aware scanning are **agent** features, not native `scan`/`run`
phases. Use `vigolium agent audit --source <path-or-git-url>` (security code audit) or
`vigolium agent query --source <path> -t code-review`. `--source` accepts a local
directory, a git URL (cloned automatically), a `.zip`/`.tar.gz`, or a `gs://` archive.

---

## Replacing standalone recon tools

A coding agent will instinctively reach for `nuclei`, `ffuf`, `katana`, `httpx`,
`gau`. Each maps to a vigolium phase (`run <phase>`) or a `kit` primitive - and
the win is that instead of piping four uninstalled binaries together, one binary
runs the phases into one DB you query with `-j/--json`.

**Do not go hunting the host for those binaries.** You will often *find* them,
and then the run's whole mapping happens outside the pinned traffic DB, outside
the phase model, with flags nobody scoped - invisible to every later `finding`,
`traffic` and `replay` call.

### Tool shims: type the argv you already know

You do not have to remember the native spelling. Vigolium accepts each tool's
own command line and routes it:

```bash
$ vigolium ffuf -u https://target.example/FUZZ -w words.txt
◆ routed to: vigolium run discovery -t https://target.example --discovery-wordlist words.txt
```

| Shim | Routes to |
|---|---|
| `vigolium ffuf -u <url>/FUZZ -w <list>` | `run discovery --discovery-wordlist` |
| `vigolium nuclei -u <url> -severity high -tags cve` | `run known-issue-scan --known-issue-scan-*` |
| `vigolium katana -u <url>` | `run spidering` |
| `vigolium gau <domain>` | `run external-harvest` |
| `vigolium arjun -u <url>` | `fuzz --fuzz param-name --anomaly` |

Mapped flags include the pace knobs (`-t`/`-threads`/`-c` -> `--concurrency`,
`-rate`/`-rl` -> `--rate-limit`), headers, proxy, and each tool's own selectors.
Both `-u X` and `-u=X` parse.

Two behaviours to expect:

- **An unmapped argument is a hard error naming the native command**, never a
  silent drop - a dropped flag is a scan that ran with a scope nobody chose.
  `vigolium ffuf --mc 200` tells you to run `vigolium run discovery --help`.
- **Where a concept does not exist, the shim says so rather than approximating.**
  `katana -d 3` errors: vigolium's spider is a state machine over DOM snapshots,
  not a depth-limited link follower, so bound it with `--spider-max-time`
  instead.

The shims are a redirect for muscle memory, not a full port. Once you know the
native form, prefer it - it takes every vigolium flag, the shims only a subset.

### The pattern

```bash
# One phase, throwaway DB, portable output - the direct tool substitute.
vigolium run <phase> -t https://target.example -S --format jsonl,fs -o out

# Or the full pipeline, which chains them (harvest -> discover -> spider ->
# known-issue-scan -> dynamic-assessment) per --strategy.
vigolium scan -t https://target.example --strategy deep
```

### nuclei -> known-issue-scan

`run known-issue-scan` *is* the nuclei engine plus native CVE modules and
Kingfisher secret scanning. Templates filter the same way:

```bash
vigolium run known-issue-scan -t https://target.example \
  --known-issue-scan-tags cve,exposure \
  --known-issue-scan-severities critical,high \
  --known-issue-scan-templates-dir ./my-templates
```

`-I nuclei -i templates/` instead *ingests* a template's request definitions as
scan traffic (it does not run the templates as nuclei would).

### ffuf / feroxbuster -> fuzz (or discovery)

`vigolium fuzz` is the deliberate ffuf-style primitive - see [fuzzing.md](fuzzing.md).

```bash
vigolium fuzz https://target.example/FUZZ -w file-long --match-status-code 200,301
```

For dir/content discovery folded into a scan, use the discovery phase with a
wordlist: `vigolium run discovery --discovery-wordlist words.txt -t <url>`.

Or just type ffuf's own argv — `vigolium ffuf -u https://t/FUZZ -w words.txt`
translates to exactly that and prints what it ran (see "Tool shims" below).

### katana / gospider / hakrawler -> spidering

`run spidering` is a headless-browser, JS-aware crawl (state machine, not a
link-follower), so it reaches SPA routes a static crawler misses:

```bash
vigolium run spidering -t https://target.example -S --format fs -o crawl
```

### gau / waybackurls -> kit harvest (or external-harvest)

```bash
vigolium kit harvest target.example                      # stateless, prints URLs
vigolium run external-harvest -t https://target.example   # into the scan DB
```

Sources: Wayback, Common Crawl, AlienVault OTX, Arquivo by default; urlscan and
VirusTotal join when their keys are set under `external_harvester`.

### httpx (probe) -> passive scan + traffic query

There is **no single prober command**. Get status/title/tech/liveness by running
a passive-only scan over a host list, then reading the records back:

```bash
vigolium scan -T hosts.txt --passive-only -S --format sqlite -o probe
vigolium traffic -S --db probe.sqlite -j --compact \
  --fields url,status_code,response_title,response_content_type,technology
```

`-S` on the scan discards the temp DB after export, so the follow-up `traffic`
must read the exported file (`-S --db`), not the project DB. Passive modules
fingerprint the tech stack; the traffic rows carry `status_code`,
`response_title`, `response_content_type`, and `technology` (omitted when empty).

### subfinder / amass -> not covered

Vigolium does **not** do external passive subdomain enumeration. Bring your own
subdomain list via `-T`. `--follow-subdomains` only pulls in-scope hosts already
observed in responses (auto-on at `--intensity deep`); it is not a substitute.
