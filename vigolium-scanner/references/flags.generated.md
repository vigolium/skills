# Flag Reference (generated)

<!-- GENERATED FILE — DO NOT EDIT BY HAND.
     Regenerate with `make skill-flags` (or `vigolium skills gen-flags`).
     Every row below is read straight off the cobra command tree, so it
     always matches the binary that produced it. -->

Every flag on every command, grouped by the command that owns it.
**Global flags are listed once** and are available everywhere; each command
section lists only the flags that command adds.

This file is for lookup — grep it for a flag name rather than reading it
top to bottom. For the authoritative help of the version you have installed,
run `vigolium <command> -h`.

## Contents

- [Global Flags](#global-flags)
- [vigolium agent](#vigolium-agent)
- [vigolium agent audit](#vigolium-agent-audit)
- [vigolium agent autopilot](#vigolium-agent-autopilot)
- [vigolium agent olium](#vigolium-agent-olium)
- [vigolium agent query](#vigolium-agent-query)
- [vigolium agent session](#vigolium-agent-session)
- [vigolium agent swarm](#vigolium-agent-swarm)
- [vigolium agent triage](#vigolium-agent-triage)
- [vigolium audit](#vigolium-audit)
- [vigolium auth lint](#vigolium-auth-lint)
- [vigolium auth list](#vigolium-auth-list)
- [vigolium auth load](#vigolium-auth-load)
- [vigolium auth totp](#vigolium-auth-totp)
- [vigolium config ls](#vigolium-config-ls)
- [vigolium db](#vigolium-db)
- [vigolium db clean](#vigolium-db-clean)
- [vigolium db export](#vigolium-db-export)
- [vigolium db list](#vigolium-db-list)
- [vigolium db stats](#vigolium-db-stats)
- [vigolium doctor](#vigolium-doctor)
- [vigolium export](#vigolium-export)
- [vigolium extensions docs](#vigolium-extensions-docs)
- [vigolium extensions eval](#vigolium-extensions-eval)
- [vigolium extensions example](#vigolium-extensions-example)
- [vigolium extensions lint](#vigolium-extensions-lint)
- [vigolium extensions ls](#vigolium-extensions-ls)
- [vigolium finding](#vigolium-finding)
- [vigolium finding load](#vigolium-finding-load)
- [vigolium fuzz](#vigolium-fuzz)
- [vigolium import](#vigolium-import)
- [vigolium ingest](#vigolium-ingest)
- [vigolium js](#vigolium-js)
- [vigolium kit harvest](#vigolium-kit-harvest)
- [vigolium kit js-beautify](#vigolium-kit-js-beautify)
- [vigolium kit jwt-crack](#vigolium-kit-jwt-crack)
- [vigolium kit oast](#vigolium-kit-oast)
- [vigolium kit oast new](#vigolium-kit-oast-new)
- [vigolium kit oast poll](#vigolium-kit-oast-poll)
- [vigolium kit payload](#vigolium-kit-payload)
- [vigolium kit secret-scan](#vigolium-kit-secret-scan)
- [vigolium log](#vigolium-log)
- [vigolium log ls](#vigolium-log-ls)
- [vigolium module](#vigolium-module)
- [vigolium module disable](#vigolium-module-disable)
- [vigolium module enable](#vigolium-module-enable)
- [vigolium module ls](#vigolium-module-ls)
- [vigolium olium](#vigolium-olium)
- [vigolium project create](#vigolium-project-create)
- [vigolium project delete](#vigolium-project-delete)
- [vigolium project list](#vigolium-project-list)
- [vigolium project use](#vigolium-project-use)
- [vigolium replay](#vigolium-replay)
- [vigolium run](#vigolium-run)
- [vigolium scan](#vigolium-scan)
- [vigolium scan-request](#vigolium-scan-request)
- [vigolium scan-url](#vigolium-scan-url)
- [vigolium server](#vigolium-server)
- [vigolium skills get](#vigolium-skills-get)
- [vigolium skills install](#vigolium-skills-install)
- [vigolium storage download](#vigolium-storage-download)
- [vigolium storage ls](#vigolium-storage-ls)
- [vigolium storage presign](#vigolium-storage-presign)
- [vigolium storage results](#vigolium-storage-results)
- [vigolium storage upload](#vigolium-storage-upload)
- [vigolium traffic](#vigolium-traffic)
- [vigolium update](#vigolium-update)

---

## Global Flags

Persistent flags available on every command.

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--ci-output-format` | — | bool | `false` | CI-friendly output: JSONL findings only, no color, no banners |
| `--config` | — | string | — | Path to config file (default "~/.vigolium/vigolium-configs.yaml") |
| `--db` | — | string | — | Path to SQLite database file (default "~/.vigolium/database-vgnm.sqlite"). Also honors $VIGOLIUM_DB_PATH, which additionally reads that file with project scoping off |
| `--debug` | — | bool | `false` | Enable debug-level logging (includes outgoing HTTP request lines) |
| `--dump-traffic` | — | bool | `false` | Print every HTTP request/response pair to stderr (Burp-style, bypasses logger) |
| `--ext` | — | stringArray | — | Load JavaScript extension script (repeatable) |
| `--ext-dir` | — | string | — | Override extension scripts directory |
| `--force` | — | bool | `false` | Skip confirmation prompts |
| `--format` | — | string | `console` | Output format (comma-separated for multiple): console, jsonl, html, sarif, sqlite (needs -S), fs (alias: file-system; flat traffic/finding tree) |
| `--full-example` | — | bool | `false` | Show full example commands organized by section |
| `--json` | `-j` | bool | `false` | Emit machine-readable JSON for agent/programmatic use (compact bodies; pair with --fields/--compact/--full-body on finding/traffic/db). For the bulk {type,data} stream use --format jsonl / export. |
| `--list-input-mode` | — | bool | `false` | List all supported input modes with examples |
| `--list-modules` | `-M` | bool | `false` | List all available scanner modules |
| `--log-file` | — | string | — | Write all log output to this file (JSON format) |
| `--mem-limit` | — | string | — | Soft heap ceiling (GOMEMLIMIT) for scans: empty = auto (1/3 of RAM, scaled down by -P/--parallel so all children stay under ⅔ of RAM), 'off' to disable, or an explicit size/percent like 6GiB or 50%. An existing GOMEMLIMIT env var overrides this. |
| `--no-color` | — | bool | `false` | Disable ANSI color in all output (also honored via the NO_COLOR env var) |
| `--project-name` | — | string | — | Project name to scope all operations to (must match exactly one project) |
| `--project-uuid` | — | string | — | Project UUID to scope all operations to (defaults to the default project) |
| `--proxy` | — | string | — | Route all requests through this proxy (HTTP/SOCKS5 URL) |
| `--scan-uuid` | — | string | — | Pin scan UUID for this session (use to sync results across nodes; defaults to a freshly-minted UUID) |
| `--silent` | — | bool | `false` | Suppress all output except findings |
| `--skip-dependency-check` | — | bool | `false` | Skip the first-run dependency check (chromium, nuclei templates) and stamp ~/.vigolium/initialized immediately |
| `--soft-fail` | — | bool | `false` | Always exit 0, even when a command fails (error is still printed to stderr; keeps wrapping scripts/CI from being interrupted) |
| `--verbose` | `-v` | bool | `false` | Enable verbose logging output |
| `--width` | — | int | `70` | Maximum column width for table output |

## vigolium agent

Run an agentic scan — AI-driven scanning with native scan support

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--list-agents` | — | bool | `false` | List the olium providers available for agent runs |
| `--list-templates` | — | bool | `false` | List available prompt templates |

## vigolium agent audit

Run audit and/or piolium back-to-back as a unified security audit

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agent` | — | string | — | [audit] Coding agent for the audit leg: claude or codex. Overrides the agent implied by --provider while keeping its resolved auth. No effect on the piolium leg. |
| `--api-key` | — | string | — | BYOK API key for the run (literal, $ENV_NAME, or @path). claude→ANTHROPIC_API_KEY, codex→OPENAI_API_KEY. Empty inherits agent.olium.* config. Mutually exclusive with --oauth-token / --oauth-cred-file. |
| `--clean-raw` | — | bool | `false` | [audit] Remove <source>/vigolium-results/ from the source tree after the run (the session copy is always kept). Inverts the default --keep-raw retention of the source-folder copy. No effect on the piolium leg. |
| `--commit-depth` | — | int | `1` | git clone --depth value when --source is a git URL (default 1; use 0 for full history; overrides --intensity) |
| `--driver` | — | string | `auto` | Audit driver: auto (audit; fall back to piolium when claude/codex CLI missing), both (audit then piolium), audit, or piolium (default auto) |
| `--intensity` | — | string | `balanced` | Audit intensity preset: quick, balanced, or deep |
| `--interactive` | `-i` | bool | `false` | Drop into the coding agent with the audit harness installed and drive the audit yourself (audit-only; the embedded vigolium-audit binary's -i). Skips NDJSON streaming, the AgenticScan row, and findings auto-import — results land in <source>/vigolium-results/; import them afterward with 'vigolium import'. Not valid with --driver=piolium. |
| `--keep-raw` | — | bool | `true` | [audit] Keep raw scanner output, draft findings, and intermediate workspaces under <source>/vigolium-results/ for manual review (overrides audit's deep/confirm auto-prune). On by default; the source-folder copy is also retained. Pass --clean-raw to remove it from the source tree. No effect on the piolium leg. |
| `--list-modes` | — | bool | `false` | List the available audit modes (audit's mode graph: phases, time estimate, descriptions) and exit |
| `--mode` | — | string | — | Audit mode override (overrides --intensity). Shared modes: lite, balanced, deep, revisit, confirm, merge. Driver-specific: piolium=longshot/smoke/diff/status, audit=reinvest/refresh/mock/diff/status |
| `--modes` | — | string | — | Run a chain of modes back-to-back (comma-separated, e.g. deep,refresh,confirm). Overrides --mode/--intensity. Stops on the first non-complete mode. audit runs the chain natively (--modes); piolium chains via sequential runs collapsed into one row; with driver=auto/both, modes a driver can't run are skipped on that driver's leg. |
| `--no-dedup` | — | bool | `false` | Skip the post-pass project-wide findings dedup that runs after the audit completes |
| `--no-preflight` | — | bool | `false` | Skip the pre-audit roundtrip checks for both drivers (pi+claude auth/model availability) |
| `--no-stream` | — | bool | `false` | Don't echo agent output to the console (still written to {session}/<driver>/runtime.log) |
| `--oauth-cred-file` | — | string | — | BYOK OAuth credential file path (literal or $ENV_NAME). Codex ('~/.codex/auth.json' shape). Staged under <pi-agent-dir>/auth.json with backup-and-restore for piolium runs. Mutually exclusive with --api-key / --oauth-token. |
| `--oauth-token` | — | string | — | BYOK Anthropic OAuth bearer token (literal, $ENV_NAME, or @path). Claude only — produced by 'claude setup-token'. Mutually exclusive with --api-key / --oauth-cred-file. |
| `--output` | `-o` | string | — | HTML report path for --stateless runs (default vigolium-result/vigolium-audit-report.html; supports gs://<project>/<key> and {ts}). Only applies with -S/--stateless. |
| `--output-dir` | — | string | — | Bundle directory for --stateless runs: collects the HTML report (as vigolium-audit-report.html) AND a copy of the raw vigolium-results/ output into one folder. A relative -o/--output is nested under it; an absolute path or gs:// URL wins. The source-tree copy is left in place. Supports {ts}/{project-uuid}. Only applies with -S/--stateless. |
| `--pi-model` | — | string | — | [piolium] Override pi's defaultModel (e.g. claude-opus-4-6, gemini-3.1-pro) |
| `--pi-provider` | — | string | — | [piolium] Override pi's defaultProvider (e.g. vertex-anthropic, google-vertex) |
| `--plm-command-retries` | — | int | `0` | [piolium] Per-command retry count (0=piolium default) |
| `--plm-longshot-langs` | — | string | — | [piolium] Longshot language allowlist (comma-separated, e.g. python,go) |
| `--plm-longshot-limit` | — | int | `0` | [piolium] Max files hunted in longshot mode (0=piolium default) |
| `--plm-longshot-timeout` | — | int | `0` | [piolium] Per-file kill timer in longshot mode in ms (0=piolium default) |
| `--plm-phase-retries` | — | int | `0` | [piolium] Per-phase retry count (0=piolium default) |
| `--plm-scan-limit` | — | int | `0` | [piolium] Cap commit-history scan to N commits (0=piolium default) |
| `--plm-scan-since` | — | string | — | [piolium] Cap commit-history scan to a git --since window (e.g. "60 days ago") |
| `--preflight-timeout` | — | duration | `30s` | Per-driver preflight timeout (e.g. 30s, 1m); applies to both pi and claude |
| `--provider` | — | string | — | [audit] Olium provider hint to drive audit's --agent: anthropic-* → claude, openai-* → codex (also forwards that provider's BYOK auth). Empty inherits agent.olium.provider. For a pure agent switch without changing auth, prefer --agent. |
| `--show-thinking` | — | bool | `false` | Render the agent's internal thinking blocks (audit NDJSON 'thinking' events) in the live stream. Off by default — thinking is verbose and produces many lines per phase. |
| `--source` | — | string | `.` | Source: local directory, git URL, gs://<project>/<key> archive, or local .zip/.tar.gz |
| `--stateless` | `-S` | bool | `false` | Run the audit into a throwaway temporary database (the main DB is left untouched) and auto-write a self-contained HTML report. Mirrors 'vigolium scan -S'. Not valid with --interactive. |
| `--upload-results` | — | bool | `false` | Upload session bundle to cloud storage after completion (requires storage config) |

## vigolium agent autopilot

Agentic scan: autonomous AI-driven vulnerability scanning

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--audit` | — | string | `lite` | vigolium-audit mode: lite (3-phase), balanced (9-phase), deep (12-phase), mock (sample output), or off (disable). Default: lite when --source is set |
| `--burp-bridge-url` | `-B` | string | — | Pull live Burp/Caido proxy history into the project DB before the run (e.g. http://127.0.0.1:9009), so the pre-scan and operator can mine it alongside prior traffic. Also honors $VIGOLIUM_BURP_BRIDGE_URL (alias: --caido-bridge-url) |
| `--db-isolate` | — | bool | `false` | Run into a private temporary database, then merge results into --db (or the default DB) at the end — lets parallel runs share one --db without write contention (SQLite only) |
| `--diff` | — | string | — | Focus on changed code: PR URL (github.com/.../pull/123), git ref range (main...branch), or HEAD~N |
| `--disable-guardrail` | — | bool | `false` | Skip the prompt-safety classifier on the natural-language prompt (use only when refusing a known-good prompt) |
| `--dry-run` | — | bool | `false` | Render the system prompt without launching the agent |
| `--files` | — | stringSlice | — | Specific files to include (relative to --source) |
| `--input` | — | string | — | Raw input (curl command, raw HTTP, Burp XML, URL). Reads from stdin if piped |
| `--intensity` | — | string | `balanced` | Scan intensity preset: quick, balanced, or deep |
| `--knowledge-base` | — | string | — | Path to a file or directory describing the app. Prose docs (markdown/txt/rst/…) are LLM-distilled into a compact brief + document index front-loaded into the operator (full docs stay on disk, read on demand). HTTP-traffic exports in the same path (HAR, Burp XML, curl, OpenAPI/Swagger, Postman, URL lists, raw HTTP) are auto-detected, parsed, and ingested into the project DB as normal traffic (source=knowledge-base) with a sample folded into the brief — disable with --knowledge-base-no-traffic. Works blackbox and whitebox. |
| `--knowledge-base-no-traffic` | — | bool | `false` | Do not parse HTTP-traffic-format files (HAR, Burp XML, curl, OpenAPI/Swagger, Postman, URL lists, raw HTTP) found in --knowledge-base into normal traffic; treat every file as prose docs instead. By default such files are parsed and ingested into the project DB (source=knowledge-base). No-op without --knowledge-base. |
| `--knowledge-base-raw` | — | bool | `false` | Skip the LLM distillation of --knowledge-base: front-load a deterministic document index only (offline / reproducible). No-op without --knowledge-base. |
| `--last-commits` | — | int | `0` | Focus on last N commits (shorthand for --diff HEAD~N) |
| `--llm-api-key` | — | string | — | Olium API key for key-based providers (falls back to agent.olium.llm_api_key or provider env var) |
| `--max-duration` | — | duration | `6h0m0s` | Maximum wall-clock duration for the autopilot session (e.g. 1h, 6h) |
| `--model` | — | string | — | Olium model id override (falls back to agent.olium.model) |
| `--no-post-halt-verify` | — | bool | `false` | Skip the post-halt coverage verification re-entry (operator halts → coverage probe → re-prompt agent when new routes turn up) |
| `--no-preflight-discovery` | — | bool | `false` | Skip the pre-flight discovery + OpenAPI/Swagger ingestion pass that seeds http_records before the operator agent starts |
| `--no-prescan` | — | bool | `false` | Skip the native pre-scan that seeds http_records before the operator agent (target-only runs; no-op when --source is set) |
| `--no-skill-filter` | — | bool | `false` | Load the full skill set; skip the pre-flight skill selection |
| `--oauth-cred` | — | string | — | Olium OAuth/SA credential file (openai-codex-oauth, anthropic-vertex, or google-vertex; falls back to agent.olium.oauth_cred_path or $GOOGLE_APPLICATION_CREDENTIALS) |
| `--oauth-token` | — | string | — | Olium Anthropic OAuth bearer token (anthropic-oauth provider; falls back to agent.olium.oauth_token or $ANTHROPIC_API_KEY) |
| `--output` | `-o` | string | — | Output base path for the --stateless export; each --format appends its own extension (report.html, report.sqlite). Defaults to vigolium-result/vigolium-autopilot. Only applies with -S/--stateless. |
| `--piolium` | — | string | — | Piolium audit mode: lite, balanced, deep, longshot, etc. Default: empty triggers auto-pick (piolium when pi is installed, else audit). Set explicitly to force piolium; set --audit=off alongside to disable audit |
| `--plan-file` | — | string | — | Path to a plan file mixing free-text guidance and raw HTTP request(s). Owns the instruction + seed input; cannot be combined with --input or a prompt (--prompt / positional) |
| `--post-halt-gap-threshold` | — | int | `0` | Minimum new (method, URL) routes the post-halt probe must turn up before the agent is re-entered. 0 = built-in default (5) |
| `--prior-context` | — | string | `auto` | Front-load a bounded summary of the traffic + findings already in the project DB so the operator mines them instead of starting from scratch: auto (default; the bounded table when prior data exists), summary (one-line pointer), off |
| `--prompt` | — | string | — | Free-text task guidance for the agent (same as the positional [prompt]; use --plan-file for a whole plan with seed HTTP requests) |
| `--prompt-file` | — | string | — | Read task guidance from a file (same channel as --prompt — its contents flow through credential extraction, so role-tagged accounts become primary/compare sessions). Convenient for long/complex prompts; mutually exclusive with --prompt, the positional [prompt], and --plan-file |
| `--provider` | — | string | — | Olium provider override: openai-codex-oauth \| openai-api-key \| openai-responses \| anthropic-api-key \| anthropic-oauth \| anthropic-cli \| anthropic-claude-sdk-bridge \| anthropic-vertex \| google-vertex \| openai-compatible \| anthropic-compatible (falls back to agent.olium.provider config) |
| `--record-uuid` | — | string | — | Use an HTTP record from the database as the seed input (looked up by UUID) |
| `--resume` | — | string | — | Resume a previous durable-autopilot run by its agentic-scan UUID: reuses its session dir, project, target, and durable scratchpad/candidates; skips pre-scan and audit re-prep (requires agent.olium.autopilot_mode != legacy) |
| `--session-dir` | — | string | — | Explicit session directory for this run's debug artifacts (transcript.jsonl, runtime.log, scratchpad, tool-results). Default: <agent.sessions_dir>/<run-uuid>. Pin it to know exactly where to look when debugging (e.g. alongside -S/--stateless scans). |
| `--show-prompt` | — | bool | `false` | Print rendered prompt to stderr before executing |
| `--skill` | — | stringSlice | — | Force-load these skills by name, bypassing the pre-flight selection (repeatable or comma-separated) |
| `--skill-tag` | — | stringSlice | — | Force-load every skill carrying one of these tags (e.g. xss,idor) |
| `--source` | — | string | — | Path to application source code for source-aware scanning |
| `--stateless` | `-S` | bool | `false` | Run the whole autopilot into a throwaway temporary database (your project DB is left untouched), then materialize --format outputs from it. Mirrors 'vigolium scan -S'. Not valid with --db, --db-isolate, or --resume. |
| `--system-prompt` | — | string | — | Replace the built-in autopilot system prompt with this value (full replace; browser section is not auto-appended) |
| `--system-prompt-file` | — | string | — | Path to a file whose contents replace the built-in autopilot system prompt (takes precedence over --system-prompt) |
| `--target` | `-t` | string | — | Target URL (derived from --input if not set) |
| `--transcript` | — | string | — | After the run, also copy the session's transcript.jsonl to this path. The in-session copy is always kept; this is a convenience for debugging (e.g. keep the transcript when the DB is throwaway). |
| `--triage` | — | bool | `false` | After the scan completes, run an AI triage pass over the findings (confirm real issues vs false positives, written back to finding status) |
| `--upload-results` | — | bool | `false` | Upload scan results to cloud storage after completion (requires storage config) |

## vigolium agent olium

Launch vigolium — interactive TUI agent (olium engine)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--base-url` | — | string | — | Endpoint URL for openai-compatible provider (e.g. http://localhost:11434/v1 for Ollama); falls back to agent.olium.custom_provider.base_url |
| `--bridge-bin` | — | string | — | Path to the 'vigolium-audit' binary hosting the SDK bridge (anthropic-claude-sdk-bridge provider; default: embedded blob, then PATH) |
| `--claude-bin` | — | string | — | Path to the 'claude' binary (anthropic-cli provider) |
| `--gcp-location` | — | string | — | GCP region for Vertex providers (else $GOOGLE_CLOUD_LOCATION, then YAML, then us-central1) |
| `--gcp-project` | — | string | — | GCP project for Vertex providers (else $GOOGLE_CLOUD_PROJECT, then YAML, then SA file's project_id) |
| `--llm-api-key` | — | string | — | API key for key-based providers (anthropic-api-key, openai-api-key); else uses ANTHROPIC_API_KEY / OPENAI_API_KEY env |
| `--model` | — | string | — | Model id (provider-specific default if empty) |
| `--oauth-cred` | — | string | — | Path to OAuth/SA credential file (openai-codex-oauth: ~/.codex/auth.json; anthropic-vertex/google-vertex: SA JSON or $GOOGLE_APPLICATION_CREDENTIALS) |
| `--oauth-token` | — | string | — | Anthropic OAuth bearer token (anthropic-oauth; falls back to agent.olium.oauth_token or $ANTHROPIC_API_KEY) |
| `--prompt` | `-p` | string | — | Run one prompt non-interactively and stream to stdout (skips the TUI). Pass '-' to read the prompt from stdin |
| `--provider` | — | string | — | Olium provider override: openai-codex-oauth \| openai-api-key \| openai-responses \| anthropic-api-key \| anthropic-oauth \| anthropic-cli \| anthropic-claude-sdk-bridge \| anthropic-vertex \| google-vertex \| openai-compatible \| anthropic-compatible (falls back to agent.olium.provider config); default openai-compatible |
| `--stdin` | — | bool | `false` | Force reading prompt from stdin |
| `--system` | — | string | — | Override system prompt |

## vigolium agent query

Send a prompt to an AI agent and get a response

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agent-label` | — | string | — | Label recorded on the AgenticScan DB row; agent dispatch always uses olium |
| `--append` | — | string | — | Append extra text to the rendered prompt |
| `--base-url` | — | string | — | Endpoint URL for openai-compatible provider (e.g. http://localhost:11434/v1); falls back to agent.olium.custom_provider.base_url |
| `--dry-run` | — | bool | `false` | Print the rendered prompt without executing |
| `--files` | — | stringSlice | — | Specific files to include (relative to --source) |
| `--gcp-location` | — | string | — | GCP region for Vertex providers (else $GOOGLE_CLOUD_LOCATION, then YAML, then us-central1) |
| `--gcp-project` | — | string | — | GCP project for Vertex providers (else $GOOGLE_CLOUD_PROJECT, then YAML, then SA file's project_id) |
| `--instruction` | — | string | — | Custom instruction to guide the agent (appended to prompt) |
| `--instruction-file` | — | string | — | Path to a file containing custom instructions |
| `--llm-api-key` | — | string | — | Olium API key for key-based providers (falls back to agent.olium.llm_api_key or provider env var) |
| `--max-duration` | — | duration | `5m0s` | Maximum wall-clock time for agent execution (0 = no limit) |
| `--model` | — | string | — | Olium model id override (falls back to agent.olium.model) |
| `--oauth-cred` | — | string | — | Olium OAuth/SA credential file (openai-codex-oauth, anthropic-vertex, or google-vertex; falls back to agent.olium.oauth_cred_path or $GOOGLE_APPLICATION_CREDENTIALS) |
| `--oauth-token` | — | string | — | Olium Anthropic OAuth bearer token (anthropic-oauth provider; falls back to agent.olium.oauth_token or $ANTHROPIC_API_KEY) |
| `--output` | — | string | — | Write agent output to this file |
| `--prompt` | `-p` | string | — | Prompt text to send to the agent |
| `--prompt-file` | — | string | — | Path to a prompt template file |
| `--prompt-template` | — | string | — | Prompt template ID (e.g. security-code-review) |
| `--provider` | — | string | — | Olium provider override: openai-codex-oauth \| openai-api-key \| openai-responses \| anthropic-api-key \| anthropic-oauth \| anthropic-cli \| anthropic-claude-sdk-bridge \| anthropic-vertex \| google-vertex \| openai-compatible \| anthropic-compatible (falls back to agent.olium.provider config) |
| `--show-prompt` | — | bool | `false` | Print rendered prompt to stderr before executing |
| `--source` | — | string | — | Path to source code repository |
| `--source-label` | — | string | — | Label for records ingested from agent output (e.g. 'agent-review') |
| `--stdin` | — | bool | `false` | Read prompt from stdin |
| `--upload-results` | — | bool | `false` | Upload session bundle to cloud storage after completion (requires storage config) |

## vigolium agent session

List agent run sessions or show session details

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--full` | — | bool | `false` | Show full raw output (shortcut for --tail -1) |
| `--limit` | `-n` | int | `50` | Maximum number of records to display |
| `--mode` | — | string | — | Filter by mode (query, autopilot, pipeline, swarm) |
| `--no-tui` | — | bool | `false` | Force TUI off (escape hatch if TUI ever becomes default) |
| `--offset` | — | int | `0` | Number of records to skip |
| `--tail` | — | int | `50` | Number of raw output lines to show (0=none, -1=all) |
| `--tui` | — | bool | `false` | Open interactive TUI (arrow keys to navigate, enter to view details, c to copy id) |

## vigolium agent swarm

Agentic scan: AI-guided targeted vulnerability swarm

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agent-label` | — | string | — | Label recorded on the AgenticScan DB row; agent dispatch always uses olium |
| `--all-records` | — | bool | `false` | Use every HTTP record in the active project as input |
| `--audit` | — | string | — | Run background vigolium-audit for parallel security auditing: 'lite' (3-phase, default), 'balanced' (9-phase), or 'deep' (12-phase). Requires --source |
| `--auth-config` | — | string | — | Path to an existing auth-config.yaml. Skips both browser auth and --cookie/--header/--login-curl synthesis. |
| `--base-url` | — | string | — | Endpoint URL for openai-compatible / anthropic-compatible provider (e.g. http://localhost:11434/v1); falls back to agent.olium.custom_provider.base_url |
| `--batch-concurrency` | — | int | `0` | Max parallel master agent batches (0 = auto, scales with CPU count) |
| `--browser-auth` | — | bool | `false` | Run the browser-based auth phase before discovery |
| `--code-audit` | — | bool | `false` | Enable AI security code audit phase (on by default when --source is provided, use --code-audit=false to disable) |
| `--cookie` | — | stringArray | — | Session cookie name=value pair (repeatable; e.g. --cookie 'session=abc123'). Injected into recon, discovery, and scan as Cookie: header. Commas are literal. |
| `--db-isolate` | — | bool | `false` | Run into a private temporary database, then merge results into --db (or the default DB) at the end — lets parallel runs share one --db without write contention (SQLite only) |
| `--diff` | — | string | — | Focus on changed code: PR URL (github.com/.../pull/123), git ref range (main...branch), or HEAD~N |
| `--disable-guardrail` | — | bool | `false` | Skip the prompt-safety classifier on the natural-language prompt (use only when refusing a known-good prompt) |
| `--discover` | — | bool | `false` | Run discovery+spidering before master agent planning to expand attack surface |
| `--dry-run` | — | bool | `false` | Render prompts without executing |
| `--files` | — | stringSlice | — | Specific source files to include (relative to --source) |
| `--gcp-location` | — | string | — | GCP region for Vertex providers (else $GOOGLE_CLOUD_LOCATION, then YAML, then us-central1) |
| `--gcp-project` | — | string | — | GCP project for Vertex providers (else $GOOGLE_CLOUD_PROJECT, then YAML, then SA file's project_id) |
| `--header` | `-H` | stringArray | — | Inject HTTP header into recon, discovery, and scan (repeatable; e.g. -H 'Authorization: Bearer xxx'). Commas are literal. |
| `--input` | — | string | — | Raw input (curl command, raw HTTP, Burp XML, URL). Reads from stdin if piped |
| `--intensity` | — | string | `balanced` | Scan intensity preset: quick, balanced, or deep |
| `--last-commits` | — | int | `0` | Focus on last N commits (shorthand for --diff HEAD~N) |
| `--llm-api-key` | — | string | — | Olium API key for key-based providers (falls back to agent.olium.llm_api_key or provider env var) |
| `--login-curl` | — | string | — | Curl command for login flow; replayed by the auth runtime to capture a fresh session. Cookies/headers from a successful response are reused for the scan. |
| `--master-batch-size` | — | int | `0` | Max records per master agent batch (0 = default 5) |
| `--max-duration` | — | duration | `12h0m0s` | Maximum swarm duration (0 = unlimited; e.g. 6h, 24h) |
| `--max-iterations` | — | int | `3` | Maximum triage-rescan iterations |
| `--max-master-retries` | — | int | `3` | Max master agent retries on parse failure |
| `--max-plan-records` | — | int | `25` | Max records sent to plan agent (selects most interesting with one slot per URL prefix; 0 = no limit). Defaults are overridden by --intensity: quick=10, balanced=25, deep=50. |
| `--max-probe-body` | — | int | `0` | Max response body size in bytes during probing (0 = default 2MB) |
| `--model` | — | string | — | Olium model id override (falls back to agent.olium.model) |
| `--modules` | `-m` | stringSlice | — | Explicit module names to include |
| `--no-skill-filter` | — | bool | `false` | Load the full skill set into triage; ignore planner selection |
| `--oauth-cred` | — | string | — | Olium OAuth/SA credential file (openai-codex-oauth, anthropic-vertex, or google-vertex; falls back to agent.olium.oauth_cred_path or $GOOGLE_APPLICATION_CREDENTIALS) |
| `--oauth-token` | — | string | — | Olium Anthropic OAuth bearer token (anthropic-oauth; falls back to agent.olium.oauth_token or $ANTHROPIC_API_KEY) |
| `--only` | — | string | — | Run only this scanning phase (discovery, spidering, spa, dynamic-assessment, external-harvest) |
| `--piolium` | — | string | — | Run background piolium audit (Pi runtime): lite, balanced, deep, longshot, etc. Requires --source. Empty triggers auto-pick when --audit is also empty (piolium when pi is installed, else nothing) |
| `--plan-file` | — | string | — | Path to a plan file mixing free-text guidance and raw HTTP request(s). Every request block becomes a seed input; cannot be combined with --input or a prompt (--prompt / positional) |
| `--probe-concurrency` | — | int | `0` | Max parallel probe requests (0 = default 10) |
| `--probe-timeout` | — | duration | `0s` | Per-request probe timeout (0 = default 10s) |
| `--profile` | — | string | — | Scanning profile to use |
| `--prompt` | — | string | — | Free-text task guidance for the agent (same as the positional [prompt]; use --plan-file for a whole plan with seed HTTP requests) |
| `--prompt-file` | — | string | — | Read task guidance from a file (same channel as --prompt). Convenient for long/complex prompts; mutually exclusive with --prompt, the positional [prompt], and --plan-file |
| `--provider` | — | string | — | Olium provider override: openai-codex-oauth \| openai-api-key \| openai-responses \| anthropic-api-key \| anthropic-oauth \| anthropic-cli \| anthropic-claude-sdk-bridge \| anthropic-vertex \| google-vertex \| openai-compatible \| anthropic-compatible (falls back to agent.olium.provider config) |
| `--record-uuid` | — | stringSlice | — | HTTP record UUID from database (repeatable, or comma-separated) |
| `--records-from` | — | string | — | Filter ingested HTTP records by spec (e.g. "host=example.com,status=200,method=GET,path=/api,since=2026-04-01") |
| `--show-prompt` | — | bool | `false` | Print rendered prompts to stderr before executing |
| `--skill` | — | stringSlice | — | Force-load these skills by name into triage, bypassing planner selection (repeatable or comma-separated) |
| `--skill-tag` | — | stringSlice | — | Force-load every skill carrying one of these tags into triage (e.g. xss,idor) |
| `--skip` | — | stringSlice | — | Skip specific phases (recon, discovery, spidering, spa, dynamic-assessment, external-harvest, triage, rescan) |
| `--source` | — | string | — | Path to application source code for route discovery |
| `--source-analysis-only` | — | bool | `false` | Run only the source analysis phase and exit |
| `--start-from` | — | string | — | Resume from a specific phase (native-normalize, source-analysis, code-audit, native-discover, native-recon, plan, native-extension, native-scan, triage) |
| `--sub-agent-concurrency` | — | int | `3` | Max parallel source analysis sub-agents (routes, auth, extensions) |
| `--target` | `-t` | string | — | Target URL (required when --source is used) |
| `--triage` | — | bool | `false` | Enable AI triage and rescan phases. Intensity preset overrides this: balanced/deep enable triage by default; quick disables it. Use --triage=false to force-disable on balanced/deep. |
| `--upload-results` | — | bool | `false` | Upload scan results to cloud storage after completion (requires storage config) |
| `--vuln-type` | — | string | — | Vulnerability type focus (e.g. sqli, xss, ssrf) |
| `--with-extensions` | — | bool | `false` | Force the extension agent to run even when the planner decides built-in modules are sufficient (no effect with --dry-run) |

## vigolium agent triage

Confirm a single finding with an AI triager; downgrade severity to info on false_positive

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--base-url` | — | string | — | Endpoint URL for openai-compatible / anthropic-compatible provider (e.g. http://localhost:11434/v1); falls back to agent.olium.custom_provider.base_url |
| `--dry-run` | — | bool | `false` | Render the triage prompt and exit without calling the agent or writing to DB |
| `--gcp-location` | — | string | — | GCP region for Vertex providers (else $GOOGLE_CLOUD_LOCATION, then YAML, then us-central1) |
| `--gcp-project` | — | string | — | GCP project for Vertex providers (else $GOOGLE_CLOUD_PROJECT, then YAML, then SA file's project_id) |
| `--llm-api-key` | — | string | — | Olium API key for key-based providers (falls back to agent.olium.llm_api_key or provider env var) |
| `--max-duration` | — | duration | `5m0s` | Maximum wall-clock time for the triage run (0 = no limit) |
| `--model` | — | string | — | Olium model id override (falls back to agent.olium.model) |
| `--oauth-cred` | — | string | — | Olium OAuth/SA credential file (openai-codex-oauth, anthropic-vertex, or google-vertex; falls back to agent.olium.oauth_cred_path or $GOOGLE_APPLICATION_CREDENTIALS) |
| `--oauth-token` | — | string | — | Olium Anthropic OAuth bearer token (anthropic-oauth; falls back to agent.olium.oauth_token or $ANTHROPIC_API_KEY) |
| `--provider` | — | string | — | Olium provider override: openai-codex-oauth \| openai-api-key \| openai-responses \| anthropic-api-key \| anthropic-oauth \| anthropic-cli \| anthropic-claude-sdk-bridge \| anthropic-vertex \| google-vertex \| openai-compatible \| anthropic-compatible (falls back to agent.olium.provider config) |
| `--show-prompt` | — | bool | `false` | Print rendered prompt to stderr before executing |

## vigolium audit

Run a unified security audit (alias for `vigolium agent audit`)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agent` | — | string | — | [audit] Coding agent for the audit leg: claude or codex. Overrides the agent implied by --provider while keeping its resolved auth. No effect on the piolium leg. |
| `--api-key` | — | string | — | BYOK API key for the run (literal, $ENV_NAME, or @path). claude→ANTHROPIC_API_KEY, codex→OPENAI_API_KEY. Empty inherits agent.olium.* config. Mutually exclusive with --oauth-token / --oauth-cred-file. |
| `--clean-raw` | — | bool | `false` | [audit] Remove <source>/vigolium-results/ from the source tree after the run (the session copy is always kept). Inverts the default --keep-raw retention of the source-folder copy. No effect on the piolium leg. |
| `--commit-depth` | — | int | `1` | git clone --depth value when --source is a git URL (default 1; use 0 for full history; overrides --intensity) |
| `--driver` | — | string | `auto` | Audit driver: auto (audit; fall back to piolium when claude/codex CLI missing), both (audit then piolium), audit, or piolium (default auto) |
| `--intensity` | — | string | `balanced` | Audit intensity preset: quick, balanced, or deep |
| `--interactive` | `-i` | bool | `false` | Drop into the coding agent with the audit harness installed and drive the audit yourself (audit-only; the embedded vigolium-audit binary's -i). Skips NDJSON streaming, the AgenticScan row, and findings auto-import — results land in <source>/vigolium-results/; import them afterward with 'vigolium import'. Not valid with --driver=piolium. |
| `--keep-raw` | — | bool | `true` | [audit] Keep raw scanner output, draft findings, and intermediate workspaces under <source>/vigolium-results/ for manual review (overrides audit's deep/confirm auto-prune). On by default; the source-folder copy is also retained. Pass --clean-raw to remove it from the source tree. No effect on the piolium leg. |
| `--list-modes` | — | bool | `false` | List the available audit modes (audit's mode graph: phases, time estimate, descriptions) and exit |
| `--mode` | — | string | — | Audit mode override (overrides --intensity). Shared modes: lite, balanced, deep, revisit, confirm, merge. Driver-specific: piolium=longshot/smoke/diff/status, audit=reinvest/refresh/mock/diff/status |
| `--modes` | — | string | — | Run a chain of modes back-to-back (comma-separated, e.g. deep,refresh,confirm). Overrides --mode/--intensity. Stops on the first non-complete mode. audit runs the chain natively (--modes); piolium chains via sequential runs collapsed into one row; with driver=auto/both, modes a driver can't run are skipped on that driver's leg. |
| `--no-dedup` | — | bool | `false` | Skip the post-pass project-wide findings dedup that runs after the audit completes |
| `--no-preflight` | — | bool | `false` | Skip the pre-audit roundtrip checks for both drivers (pi+claude auth/model availability) |
| `--no-stream` | — | bool | `false` | Don't echo agent output to the console (still written to {session}/<driver>/runtime.log) |
| `--oauth-cred-file` | — | string | — | BYOK OAuth credential file path (literal or $ENV_NAME). Codex ('~/.codex/auth.json' shape). Staged under <pi-agent-dir>/auth.json with backup-and-restore for piolium runs. Mutually exclusive with --api-key / --oauth-token. |
| `--oauth-token` | — | string | — | BYOK Anthropic OAuth bearer token (literal, $ENV_NAME, or @path). Claude only — produced by 'claude setup-token'. Mutually exclusive with --api-key / --oauth-cred-file. |
| `--output` | `-o` | string | — | HTML report path for --stateless runs (default vigolium-result/vigolium-audit-report.html; supports gs://<project>/<key> and {ts}). Only applies with -S/--stateless. |
| `--output-dir` | — | string | — | Bundle directory for --stateless runs: collects the HTML report (as vigolium-audit-report.html) AND a copy of the raw vigolium-results/ output into one folder. A relative -o/--output is nested under it; an absolute path or gs:// URL wins. The source-tree copy is left in place. Supports {ts}/{project-uuid}. Only applies with -S/--stateless. |
| `--pi-model` | — | string | — | [piolium] Override pi's defaultModel (e.g. claude-opus-4-6, gemini-3.1-pro) |
| `--pi-provider` | — | string | — | [piolium] Override pi's defaultProvider (e.g. vertex-anthropic, google-vertex) |
| `--plm-command-retries` | — | int | `0` | [piolium] Per-command retry count (0=piolium default) |
| `--plm-longshot-langs` | — | string | — | [piolium] Longshot language allowlist (comma-separated, e.g. python,go) |
| `--plm-longshot-limit` | — | int | `0` | [piolium] Max files hunted in longshot mode (0=piolium default) |
| `--plm-longshot-timeout` | — | int | `0` | [piolium] Per-file kill timer in longshot mode in ms (0=piolium default) |
| `--plm-phase-retries` | — | int | `0` | [piolium] Per-phase retry count (0=piolium default) |
| `--plm-scan-limit` | — | int | `0` | [piolium] Cap commit-history scan to N commits (0=piolium default) |
| `--plm-scan-since` | — | string | — | [piolium] Cap commit-history scan to a git --since window (e.g. "60 days ago") |
| `--preflight-timeout` | — | duration | `30s` | Per-driver preflight timeout (e.g. 30s, 1m); applies to both pi and claude |
| `--provider` | — | string | — | [audit] Olium provider hint to drive audit's --agent: anthropic-* → claude, openai-* → codex (also forwards that provider's BYOK auth). Empty inherits agent.olium.provider. For a pure agent switch without changing auth, prefer --agent. |
| `--show-thinking` | — | bool | `false` | Render the agent's internal thinking blocks (audit NDJSON 'thinking' events) in the live stream. Off by default — thinking is verbose and produces many lines per phase. |
| `--source` | — | string | `.` | Source: local directory, git URL, gs://<project>/<key> archive, or local .zip/.tar.gz |
| `--stateless` | `-S` | bool | `false` | Run the audit into a throwaway temporary database (the main DB is left untouched) and auto-write a self-contained HTML report. Mirrors 'vigolium scan -S'. Not valid with --interactive. |
| `--upload-results` | — | bool | `false` | Upload session bundle to cloud storage after completion (requires storage config) |

## vigolium auth lint

Validate session auth config files for errors and warnings

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--stdin` | — | bool | `false` | Read session config from stdin |

## vigolium auth list

List session authentication configs

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--host` | — | string | — | Filter by hostname |

## vigolium auth load

Load session auth configs from a file or stdin into the database

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agent-format` | — | bool | `false` | Force parsing as agent session-config.json format |
| `--host` | — | string | — | Hostname to associate sessions with (derived from login URL if omitted) |
| `--name` | — | string | — | Session name (used with raw HTTP request input) |
| `--no-validate` | — | bool | `false` | Skip executing login flows for validation |
| `--source` | — | string | `cli` | Source label for the session rows |

## vigolium auth totp

Generate a TOTP code from a base32 secret

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--secret` | — | string | — | Base32-encoded TOTP secret (required) |

## vigolium config ls

Display current configuration

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--show-secrets` | — | bool | `false` | Reveal sensitive values (API keys, tokens, credentials) in plaintext instead of [redacted]; prints a warning to stderr |

## vigolium db

Manage database records

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--search` | — | string | — | Quick search across record fields (URLs, paths, descriptions) |
| `--table` | — | string | — | Database table to operate on (http_records, findings, scans) |
| `--watch` | — | string | — | Re-run on interval (e.g. 10s, 1m, 5m) |

## vigolium db clean

Clean database records

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--all` | — | bool | `false` | Delete every row from all data tables (requires --force) |
| `--before` | — | string | — | Delete records created strictly before this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339 (the named day itself is kept) |
| `--dry-run` | — | bool | `false` | Show what would be deleted without deleting |
| `--findings-only` | — | bool | `false` | Delete findings only, keep HTTP records |
| `--host` | — | string | — | Delete records matching the specified hostname |
| `--orphans` | — | bool | `false` | Delete findings with no matching HTTP record |
| `--severity` | — | string | — | Delete findings matching the specified severity level |
| `--status` | — | intSlice | — | Delete records with matching HTTP status codes |
| `--table` | — | string | — | Delete all rows from a specific table (e.g., http_records, findings, scans) |

## vigolium db export

Export database records

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--from` | — | string | — | Export records at or after this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339 (alias: --since) |
| `--host` | — | string | — | Filter records by hostname pattern |
| `--limit` | — | int | `0` | Maximum number of records to export, 0 for unlimited |
| `--method` | — | stringSlice | — | Filter records by HTTP method (can be specified multiple times) |
| `--offset` | — | int | `0` | Number of records to skip before exporting |
| `--output` | `-o` | string | — | Output file path, defaults to stdout |
| `--path` | — | string | — | Filter records by URL path pattern |
| `--report-url` | — | string | — | URL for the "Raw Report URL" button in HTML reports (overrides VIGOLIUM_REPORT_SHARED_URL) |
| `--request-only` | — | bool | `false` | Export only HTTP requests, omitting responses (raw format only) |
| `--severity` | — | string | — | Filter findings by severity level |
| `--status` | — | intSlice | — | Filter records by HTTP status code (can be specified multiple times) |
| `--to` | — | string | — | Export records at or before this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339; a bare date covers the whole day (alias: --until) |
| `--uuid` | — | string | — | Export a single record by its UUID |

## vigolium db list

List database records (default: http_records)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--asc` | — | bool | `false` | Sort in ascending order instead of descending |
| `--body` | — | string | — | Search within request or response body content |
| `--columns` | — | stringSlice | — | Columns to include in output, comma-separated |
| `--compact` | — | bool | `false` | With --json, emit metadata only (omit request/response bodies). --markdown already compacts response bodies by default; use --full-body to render them whole |
| `--fields` | — | stringSlice | — | Restrict --json output to these top-level keys (comma-separated, e.g. id,severity,url) |
| `--finding-source` | — | string | — | Filter findings by source (dynamic-assessment, spa, agent, oast, source-tools, extension) |
| `--from` | — | string | — | Show records at or after this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339 (alias: --since) |
| `--full-body` | — | bool | `false` | Render complete request/response bodies (no truncation/stubbing) with --json, and whole (uncompacted) bodies with --markdown |
| `--header` | — | string | — | Search within HTTP header names and values |
| `--host` | — | string | — | Filter records by hostname pattern (wildcard supported) |
| `--limit` | `-n` | int | `100` | Maximum number of records to display |
| `--list-columns` | — | bool | `false` | List column names for the current table |
| `--list-tables` | — | bool | `false` | List all database table names |
| `--method` | — | stringSlice | — | Filter records by HTTP method (can be specified multiple times) |
| `--min-risk` | — | int | `0` | Show only records with risk score at or above this value |
| `--module-type` | — | string | — | Filter findings by module type (active, passive, nuclei, agent, source-tools, oast, extension) |
| `--offset` | — | int | `0` | Number of records to skip before displaying |
| `--path` | — | string | — | Filter records by URL path pattern |
| `--raw` | — | bool | `false` | Show full raw HTTP request and response |
| `--record-kind` | — | string | — | Filter by record kind (finding, candidate, observation; comma-separated). Default: finding |
| `--remark` | — | string | — | Filter records containing this text in remarks |
| `--severity` | — | string | — | Filter findings by severity: critical,high,medium,low,suspect,info (comma-separated; single-letter shorthands ok, e.g. 'h,c') |
| `--sort` | — | string | `created_at` | Sort results by field: uuid, created_at, sent_at, method, status_code, response_time |
| `--status` | — | intSlice | — | Filter records by HTTP status code (can be specified multiple times) |
| `--to` | — | string | — | Show records at or before this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339; a bare date covers the whole day (alias: --until) |
| `--tree` | — | bool | `false` | Display results in hierarchical tree format |

## vigolium db stats

Show database statistics

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--detailed` | — | bool | `false` | Show per-host and per-module breakdown |
| `--host` | — | string | — | Show statistics for a specific hostname |

## vigolium doctor

Check system readiness and diagnose configuration issues

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--fix` | — | bool | `false` | Auto-install/fix failing checks |
| `--only` | — | stringSlice | — | Fix only specific items (nuclei,chrome,bun,claude,agent-browser,pi,piolium) |

## vigolium export

Export database tables and module registry

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--exclude` | — | stringSlice | `[module]` | Exclude items by type (comma-separated, e.g. module,scan) |
| `--glob-db` | — | string | — | Export across a glob of result files merged into one temporary DB (e.g. --glob-db 'scans/*.sqlite'); implies -S |
| `--limit` | — | int | `0` | Maximum number of records to export per table (0 = unlimited) |
| `--omit-response` | — | bool | `false` | Omit raw HTTP request/response bytes (keeps metadata, smaller files) |
| `--only` | — | stringSlice | — | Export only these tables (repeatable: http, findings, scans, modules, oast, source-repos, scopes) |
| `--output` | `-o` | string | — | Output file path or gs://<project>/<key> URL (required for html; base path when multiple formats are given); supports {ts} and {project-uuid} placeholders |
| `--report-duration` | — | string | — | Human-readable scan duration for the report (e.g. "10h42m5s") |
| `--report-generated-at` | — | string | — | ISO timestamp for report generation (e.g. "2026-04-18T03:00:00Z") |
| `--report-target` | — | string | — | Target name for the report (e.g. repository name or URL) |
| `--report-title` | — | string | — | Custom title for the HTML report (default: "Vigolium Static Report") |
| `--report-url` | — | string | — | URL for the "Raw Report URL" button in HTML reports (overrides VIGOLIUM_REPORT_SHARED_URL) |
| `--search` | — | string | — | Fuzzy search filter across URLs, paths, hostnames, methods, content types, and sources |
| `--severity` | — | string | — | Filter findings by severity (comma-separated: critical,high,medium,low,info) |
| `--stateless` | `-S` | bool | `false` | Read from --db (a standalone .sqlite or .jsonl export) instead of your project DB; never writes to it |

## vigolium extensions docs

Show extension API reference with examples

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--example` | — | bool | `false` | Show usage examples for each function |

## vigolium extensions eval

Evaluate JavaScript code with vigolium.* APIs available

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--ext-file` | — | string | — | Path to JS file to evaluate |
| `--stdin` | — | bool | `false` | Read JS code from stdin |

## vigolium extensions example

Print copy-pasteable example extensions in every supported format

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--lang` | — | string | — | Restrict to one language: javascript (js), yaml, or json |
| `--list` | `-l` | bool | `false` | Print only the catalog index (keys + titles), no code |

## vigolium extensions lint

Validate extension files for syntax errors and unknown API calls

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--stdin` | — | bool | `false` | Read extension source from stdin |

## vigolium extensions ls

List loaded extensions

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--type` | — | string | `all` | Filter by type: all, active, passive, pre_hook, post_hook |

## vigolium finding

Browse vulnerability findings with fuzzy search and filtering

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agentic-scan` | — | string | — | Filter by agentic-scan UUID (findings produced by an agent autopilot/swarm/audit run) |
| `--asc` | — | bool | `false` | Sort in ascending order (default: descending) |
| `--body` | — | string | — | Search within HTTP request/response body content |
| `--burp` | — | bool | `false` | Display in Burp Suite-style format (colored request/response) |
| `--burp-bridge-url` | `-B` | string | — | Loopback Burp/Caido bridge URL used by --push-to-burp / --to-repeater (alias: --caido-bridge-url) |
| `--columns` | — | stringSlice | — | Columns to show (comma-separated, e.g. ID,SEVERITY,MODULE) |
| `--compact` | — | bool | `false` | With --json, emit metadata only (omit request/response bodies). --markdown already compacts response bodies by default; use --full-body to render them whole |
| `--confidence` | — | string | — | Filter by confidence: certain,firm,tentative (comma-separated) |
| `--exclude-body` | — | string | — | Exclude findings whose linked request/response body contains the term (inverse of --body) |
| `--exclude-columns` | — | stringSlice | — | Columns to hide (comma-separated) |
| `--exclude-header` | — | string | — | Exclude findings whose linked request/response headers contain the term (inverse of --header) |
| `--exclude-search` | — | stringArray | — | Exclude findings where the term appears in the module metadata, matched location, or linked request/response (repeatable; dropped if ANY term matches — the inverse of --search) |
| `--fields` | — | stringSlice | — | Restrict --json output to these top-level keys (comma-separated, e.g. id,severity,url) |
| `--finding-source` | — | string | — | Filter by finding source (dynamic-assessment, spa, agent, oast, source-tools, extension) |
| `--from` | — | string | — | Show findings at or after this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339 (alias: --since) |
| `--full-body` | — | bool | `false` | Render complete request/response bodies (no truncation/stubbing) with --json, and whole (uncompacted) bodies with --markdown |
| `--glob-db` | — | string | — | Read across a glob of result files merged into one temporary DB (e.g. --glob-db 'scans/*.sqlite'); implies -S |
| `--header` | — | string | — | Search within HTTP header names and values |
| `--host` | — | string | — | Filter by hostname pattern (wildcard supported) |
| `--http-mode` | — | string | — | With --send-via-burp: wire protocol — auto\|http1\|http2\|http2_ignore_alpn (default auto) |
| `--id` | — | int | `0` | Filter by finding ID |
| `--limit` | `-n` | int | `100` | Maximum findings to display |
| `--markdown` | — | bool | `false` | Render the matched findings as Markdown (evidence + request/response in fenced http blocks) to stdout; response bodies are compacted to a preview by default (use --full-body for whole bodies) |
| `--method` | — | stringSlice | — | Filter by HTTP method (repeatable) |
| `--min-severity` | — | string | — | Filter by minimum severity (e.g. high → high+critical); ignored when --severity is set |
| `--module-type` | — | string | — | Filter by module type (active, passive, nuclei, agent, source-tools, oast, extension) |
| `--no-tui` | — | bool | `false` | Force TUI off (escape hatch if TUI ever becomes default) |
| `--offset` | — | int | `0` | Number of findings to skip (for pagination) |
| `--path` | — | string | — | Filter by URL path pattern |
| `--pick` | — | string | — | Select finding(s) by 1-based position in the result list (e.g. 2, 1,3, 2-4); applied after --search/filters and sort |
| `--push-to-burp` | — | bool | `false` | Push the selected finding(s)' evidence request+response to Burp's Organizer for manual confirmation; requires --burp-bridge-url |
| `--raw` | — | bool | `false` | Show full raw HTTP request and response for each finding |
| `--record-kind` | — | string | — | Filter by record kind (finding, candidate, observation; comma-separated). Default: finding |
| `--search` | — | stringArray | — | Search across the finding's module metadata, matched location, and the linked request/response (headers + body); repeatable, AND-combined (each term further narrows) |
| `--send-via-burp` | — | bool | `false` | With --push-to-burp/--to-repeater: re-issue the request through Burp's engine and store the fresh response |
| `--severity` | — | string | — | Filter by severity: critical,high,medium,low,suspect,info (comma-separated; single-letter shorthands or any unambiguous prefix ok, e.g. 'h,c' or 'me,info'). Alias: --sev |
| `--sort` | — | string | `found_at` | Sort by: found_at, created_at, severity, module, confidence |
| `--source` | — | string | — | Filter by record source (e.g. scanner, ingest-cli) |
| `--stateless` | `-S` | bool | `false` | Read from --db (a .jsonl export or standalone .sqlite) with project scoping off; never writes to your project DB |
| `--status` | — | intSlice | — | Filter by HTTP status code (repeatable) |
| `--to` | — | string | — | Show findings at or before this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339; a bare date covers the whole day (alias: --until) |
| `--to-repeater` | — | bool | `false` | Push to a Burp Repeater tab instead of the Organizer (respects Burp's 30-tabs/min cap) |
| `--tree` | — | bool | `false` | Display as a host/path hierarchy tree; repeated titles collapse into one node with each affected URL listed below |
| `--tui` | — | bool | `false` | Open interactive TUI (arrow keys to navigate, enter to view details, c to copy id) |
| `--with-records` | — | bool | `false` | With --json: resolve and embed the linked HTTP records (self-contained triage bundle) |

## vigolium finding load

Import findings from a file or stdin

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--finding-file` | — | string | — | Path to findings file |

## vigolium fuzz

Inject payloads into a request and report per-payload response anomalies

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--all-results` | — | bool | `false` | Emit every result, not just matched ones |
| `--anomaly` | `-a` | bool | `false` | Auto-detect interesting responses instead of requiring explicit matchers: score each response against the baseline AND the run's own population (rare status, unique body, size/time outliers, new error signatures, changed headers, unencoded reflection) |
| `--anomaly-min-population` | — | int | `0` | Responses needed before population signals (rarity/outliers) count (default 12) |
| `--anomaly-threshold` | — | string | `medium` | How strong a signal must be to report: low\|medium\|high, or a number |
| `--auth-session` | — | string | — | Auth session name whose headers are merged in (from 'vigolium auth list') |
| `--baseline-samples` | — | int | `0` | Send the un-fuzzed request this many times to measure timing jitter, enabling time_z (default 1, or 3 with --anomaly) |
| `--burp-bridge-url` | `-B` | string | — | Loopback Burp/Caido bridge URL used by --send-via-burp / --matches-to-organizer (alias: --caido-bridge-url) |
| `--cacert` | — | string | — | CA bundle used to verify the target (implies --verify-tls) |
| `--cert` | — | string | — | Client certificate file (PEM), curl's -E |
| `--class` | — | stringSlice | — | Built-in payload class to inject: cmdi,crlf,lfi,open_redirect,path_traversal,sqli,ssrf,ssti,xss,xxe (comma-list) |
| `--compressed` | — | bool | `false` | Request a compressed response (already transparent) — accepted for curl compatibility |
| `--concurrency` | `-c` | int | `10` | Concurrent requests |
| `--connect-timeout` | — | duration | `0s` | TCP/TLS connect timeout (e.g. 5s) |
| `--cookie` | — | string | — | Cookie header value, curl's -b (e.g. 'a=1; b=2') |
| `--data` | `-d` | string | — | Request body override (refreshes Content-Length) |
| `--data-binary` | — | stringArray | — | Body data; '@file' is read verbatim, newlines kept (repeatable) |
| `--data-raw` | — | stringArray | — | Body data with '@' taken literally (repeatable) |
| `--data-urlencode` | — | stringArray | — | URL-encoded body data; supports curl's name=value, @file and name@file forms (repeatable) |
| `--delay` | — | int | `0` | Delay in ms before each request (per worker) |
| `--dry-run` | — | bool | `false` | Resolve positions and payloads, print what would be sent, and exit without any network traffic |
| `--exclude-header` | — | stringArray | — | Exclude on a response header: 'Name' for presence, or 'Name: regex' (repeatable) |
| `--exclude-lines` | — | stringSlice | — | Exclude response line counts: N, N-M, >N, <N, !N |
| `--exclude-mode` | — | string | `any` | Combine exclude flags with 'any' (OR) or 'all' (AND) |
| `--exclude-regex` | — | string | — | Exclude responses whose body matches this regex |
| `--exclude-size` | — | stringSlice | — | Exclude response sizes in bytes: N, N-M, >N, <N, !N |
| `--exclude-status-code` | — | stringSlice | — | Exclude status codes: N, N-M, >N, <N, !N, or 'all' |
| `--exclude-time` | — | stringSlice | — | Exclude response times in ms: N, N-M, >N, <N, !N (bare N means >=N) |
| `--exclude-time-z` | — | stringSlice | — | Exclude response times by standard deviations above the baseline mean (needs --baseline-samples > 1); e.g. 4 or '>3' |
| `--exclude-words` | — | stringSlice | — | Exclude response word counts: N, N-M, >N, <N, !N |
| `--fail-on-match` | — | bool | `false` | Exit non-zero (3) if any result matches (for agent/CI gating) |
| `--form` | — | stringArray | — | multipart/form-data field 'name=value', 'name=@file' or 'name=<file' (repeatable) |
| `--form-string` | — | stringArray | — | multipart field whose value is never interpreted (repeatable) |
| `--fuzz` | — | string | — | What to fuzz: method\|path\|params\|param-name\|headers\|cookies\|all (default: all insertion points) |
| `--fuzz-header` | — | stringArray | — | Fuzz a specific header by name (repeatable) |
| `--get` | — | bool | `false` | Send the accumulated data as query parameters on a GET, curl's -G |
| `--head` | — | bool | `false` | Use HEAD, curl's -I |
| `--header` | `-H` | stringArray | — | Request header 'Name: value', added or replaced on the source request (repeatable) |
| `--http-mode` | — | string | — | With --send-via-burp: wire protocol — auto\|http1\|http2\|http2_ignore_alpn (default auto; use http1 for request smuggling/desync) |
| `--http-version` | — | string | — | Force a wire protocol: 1.1 or 2 (curl's --http1.1 / --http2) |
| `--http1.1` | — | bool | `false` | Force HTTP/1.1 |
| `--http2` | — | bool | `false` | Force HTTP/2 |
| `--ignore-scope` | — | bool | `false` | Fuzz a host outside the project's configured scope |
| `--input` | `-i` | string | — | Raw input: curl, raw HTTP, Burp XML, base64, URL, or '-' for stdin |
| `--input-file` | — | string | — | Read --input value from a file |
| `--insecure` | — | bool | `false` | Skip TLS verification (already the default; accepted for curl compatibility) |
| `--key` | — | string | — | Client certificate private key file (PEM) |
| `--keyword` | — | string | `FUZZ` | Marker keyword replaced by each payload when present in the request |
| `--location` | — | bool | `false` | Follow 30x redirects (already the default; use --no-redirects to stop) — accepted for curl compatibility |
| `--match-header` | — | stringArray | — | Match on a response header: 'Name' for presence, or 'Name: regex' (repeatable) |
| `--match-lines` | — | stringSlice | — | Match response line counts: N, N-M, >N, <N, !N |
| `--match-mode` | — | string | `any` | Combine match flags with 'any' (OR) or 'all' (AND) |
| `--match-regex` | — | string | — | Match responses whose body matches this regex |
| `--match-size` | — | stringSlice | — | Match response sizes in bytes: N, N-M, >N, <N, !N |
| `--match-status-code` | — | stringSlice | — | Match status codes: N, N-M, >N, <N, !N, or 'all' |
| `--match-time` | — | stringSlice | — | Match response times in ms: N, N-M, >N, <N, !N (bare N means >=N) |
| `--match-time-z` | — | stringSlice | — | Match response times by standard deviations above the baseline mean (needs --baseline-samples > 1); e.g. 4 or '>3' |
| `--match-words` | — | stringSlice | — | Match response word counts: N, N-M, >N, <N, !N |
| `--matches-to-organizer` | — | bool | `false` | Push each matched result's request to Burp's Organizer (Burp re-issues it) for manual triage; requires --burp-bridge-url |
| `--max-redirs` | — | int | `0` | Maximum redirects to follow (0 = Go's default of 10) |
| `--max-time` | — | duration | `0s` | Total per-request budget; alias for --timeout, curl's -m |
| `--mode` | — | string | `sniper` | With several markers (FUZZ, FUZZ2, ...): sniper (one at a time) \| batteringram (same payload everywhere) \| pitchfork (lists in lockstep) \| clusterbomb (every combination) |
| `--no-calibrate` | — | bool | `false` | Disable auto-calibration of the target's catch-all response |
| `--no-cookies` | — | bool | `false` | Don't carry cookies (overrides --session-id) |
| `--no-redirects` | — | bool | `false` | Don't follow 30x redirects |
| `--output` | `-o` | string | — | Write JSONL results to this file (default: stdout) |
| `--path-as-is` | — | bool | `false` | Send the path byte-for-byte (already the case; see pkg/replay wire fidelity) — accepted for curl compatibility |
| `--payload` | `-p` | stringArray | — | Inline payload literal (repeatable) |
| `--point` | — | stringArray | — | Explicit insertion point 'TYPE:name' e.g. URL_PARAM:id (repeatable) |
| `--pretty` | — | bool | `false` | Human-readable table instead of JSONL |
| `--record-uuid` | `-u` | string | — | Use a stored HTTP record (by UUID) as the request to fuzz |
| `--referer` | — | string | — | Referer header, curl's -e |
| `--request` | `-X` | string | — | HTTP method override (default: the source request's, or GET) |
| `--resolve` | — | stringArray | — | Pin a host to an address, curl's --resolve host:port:addr (repeatable) |
| `--send-timeout` | — | duration | `0s` | With --send-via-burp: per-request response timeout (<=2m; default uses the bridge's 30s) |
| `--send-via-burp` | — | bool | `false` | Send each payload through Burp's own HTTP stack (exact bytes — malformed/smuggling preserved) instead of Go's client; requires --burp-bridge-url |
| `--session-id` | — | string | — | Persist cookies across runs under ~/.vigolium/replay-jars/<id>.json |
| `--target` | `-t` | string | — | Override scheme/host/port the request is sent to (e.g. https://staging.acme.test) |
| `--timeout` | — | duration | `25s` | Per-request timeout (e.g. 30s) |
| `--user` | — | string | — | Basic auth 'user:password', curl's -u |
| `--user-agent` | — | string | — | User-Agent header, curl's -A |
| `--verify-tls` | — | bool | `false` | Verify TLS certificates — the inverse of --insecure, and NOT the default |
| `--wordlist` | `-w` | stringArray | — | Payload wordlist: a builtin name or file path (repeatable) |

## vigolium import

Import scan data, databases, or live Burp Proxy history

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--burp-bridge-url` | `-B` | string | — | Import live Burp/Caido proxy history from this loopback bridge URL into the database (alias: --caido-bridge-url) |
| `--glob-db` | — | string | — | Glob of local files to import alongside any positional paths (use one format per run), e.g. --glob-db 'prefix-*.sqlite' or '*.jsonl' |
| `--output` | `-o` | string | — | Report output path or gs://<project>/<key> URL (required when --format is set; supports {ts}) |
| `--report-duration` | — | string | — | Human-readable scan duration for the report (e.g. "10h42m5s") |
| `--report-generated-at` | — | string | — | ISO timestamp for report generation (e.g. "2026-04-18T03:00:00Z") |
| `--report-target` | — | string | — | Target name for the report (e.g. repository name or URL) |
| `--report-title` | — | string | — | Custom title for the HTML report (default: "Vigolium Static Report") |
| `--report-url` | — | string | — | URL for the "Raw Report URL" button in HTML reports (overrides VIGOLIUM_REPORT_SHARED_URL) |
| `--search` | — | string | — | Fuzzy search filter across finding fields included in the report |
| `--severity` | — | string | — | Filter report findings by severity (comma-separated: critical,high,medium,low,info) |
| `--upload` | — | bool | `false` | Upload the local import source to cloud storage after import |
| `--upload-key` | — | string | — | Explicit storage key for --upload (default: imports/<basename>-<ts>.<ext>) |

## vigolium ingest

Ingest HTTP requests into database (locally or via server)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--concurrency` | `-c` | int | `50` | Number of concurrent scan workers |
| `--disable-fetch-response` | — | bool | `false` | Store requests without fetching responses during ingestion |
| `--full-native-scan-on-receive` | — | bool | `false` | Run the full native scan pipeline (discovery + spidering + dynamic-assessment) continuously on received records, instead of dynamic-assessment only |
| `--input` | `-i` | string | `-` | Input file path or spec (use - for stdin) |
| `--input-mode` | `-I` | string | `urls` | Input format: urls, openapi, swagger, wsdl, burp, curl, nuclei, har (see --list-input-mode) |
| `--input-read-timeout` | — | duration | `3m0s` | Timeout for reading input from stdin or file |
| `--max-findings-per-module` | — | int | `10` | Stop reporting after N findings per module (0 = unlimited) |
| `--max-host-error` | — | int | `30` | Skip host after reaching this many consecutive errors |
| `--max-per-host` | — | int | `50` | Maximum concurrent requests allowed per host |
| `--module-tag` | — | stringSlice | — | Filter modules by tag (OR condition, e.g. --module-tag spring --module-tag injection) |
| `--modules` | `-m` | stringSlice | — | Scan modules to enable (default "all", supports fuzzy match on ID/name, e.g. -m xss -m sqli) |
| `--no-clustering` | — | bool | `false` | Disable deduplication of identical concurrent HTTP requests |
| `--no-tech-filter` | — | bool | `false` | Disable the tech-stack allowlist (run every module regardless of detected stack). Auto-enabled by --intensity=deep. |
| `--no-waf-pacing` | — | bool | `false` | Disable proactive CDN/WAF-edge pacing (don't pre-throttle per-host concurrency when a CloudFront/Cloudflare/etc. edge is detected); reactive back-off after a WAF block still applies |
| `--rate-limit` | `-r` | int | `100` | Global requests/second cap, enforced across native scanning and known-issue-scan when set (unset = per-host concurrency only) |
| `--scan-on-receive` | `-S` | bool | `false` | Continuously scan new HTTP records as they arrive in the database |
| `--scope-origin` | — | string | — | Host scope strictness: all, relaxed, balanced, strict |
| `--server` | `-s` | string | — | Server URL for remote ingestion (omit for local mode) |
| `--spec-default` | — | string | `1` | Fallback value for required OpenAPI parameters that lack examples |
| `--spec-header` | — | stringArray | — | Add HTTP header to OpenAPI-generated requests (repeatable; commas are literal) |
| `--spec-url` | — | bool | `false` | Use base URLs from the OpenAPI spec's servers field |
| `--spec-var` | — | stringSlice | — | Set OpenAPI parameter value as key=value (repeatable) |
| `--target` | `-t` | stringArray | — | Target URL to scan (repeatable). Commas are literal so a URL query like ?ids=1,2,3 stays one target — repeat -t for multiple targets. |
| `--target-file` | `-T` | stringArray | — | File containing target URLs (one per line; repeatable for multiple files). Commas in the path are literal. |
| `--timeout` | — | duration | `15s` | HTTP request timeout (e.g. 30s, 1m, 2h) |

## vigolium js

Execute JavaScript with the full vigolium.* API

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--code` | — | string | — | Inline JavaScript code to execute |
| `--code-file` | — | string | — | Path to JavaScript/TypeScript file to execute |
| `--target` | — | string | — | Set TARGET variable in JS scope (URL) |
| `--timeout` | — | duration | `30s` | Execution timeout |

## vigolium kit harvest

Collect known URLs for a domain from public archives and indexes

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--source` | — | stringSlice | — | Override sources (comma-separated): wayback, commoncrawl, alienvault, arquivo, urlscan, virustotal |
| `--timeout` | — | duration | `5m0s` | Overall harvest timeout |

## vigolium kit js-beautify

Unminify and unpack a JavaScript bundle into readable source

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--extract` | — | bool | `false` | Also extract endpoints/requests (a full analysis pass); included in -j output |
| `--timeout` | — | duration | `30s` | Timeout for fetching a URL argument |

## vigolium kit jwt-crack

Recover a JWT's HMAC signing secret from a wordlist

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--fail-on-crack` | — | bool | `false` | Exit 3 when the secret is recovered (for CI/agent gating) |
| `--secret` | `-p` | stringArray | — | Additional inline candidate secret (repeatable) |
| `--wordlist` | `-w` | string | — | Wordlist: a built-in name (see `kit wordlist`) or a file path (default: embedded jwt.secrets.list) |

## vigolium kit oast

Mint out-of-band (OOB/OAST) callback URLs and drain their interactions

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--session` | `-o` | string | `oast-session.yaml` | Session file path (holds the interactsh keys/correlation id) |

## vigolium kit oast new

Register an OAST session and mint one or more callback URLs

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--count` | `-n` | int | `1` | Number of callback URLs to mint |
| `--server` | — | string | `oast.pro` | interactsh server URL |
| `--token` | — | string | — | interactsh auth token (for a self-hosted server) |

## vigolium kit oast poll

Poll a saved OAST session for received interactions

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--deregister` | — | bool | `false` | After draining, deregister the session server-side and delete the session file |
| `--interval` | — | duration | `5s` | Polling interval (auto-shrunk if larger than --wait) |
| `--wait` | — | duration | `15s` | How long to poll for interactions before exiting |

## vigolium kit payload

Emit built-in payload sets by vulnerability class

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--class` | `-c` | stringSlice | — | Vulnerability class(es), comma-separated (see --list) |
| `--list` | — | bool | `false` | List available payload classes |

## vigolium kit secret-scan

Scan files or stdin for leaked credentials using the built-in secret catalog

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--exclude-rule` | — | stringSlice | — | Skip these rule IDs (comma-separated / repeatable) |
| `--fail-on-match` | — | bool | `false` | Exit 3 when at least one secret is found (for CI/agent gating) |
| `--max-file-size` | — | int64 | `16777216` | Skip files larger than this many bytes |
| `--min-confidence` | — | string | `low` | Minimum confidence to report: low, medium, high |
| `--redact` | — | bool | `false` | Mask the secret value in output (show only a prefix/suffix) |
| `--rule` | — | stringSlice | — | Only report these rule IDs (comma-separated / repeatable) |

## vigolium log

View raw logs for a native scan or agentic scan

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--follow` | `-f` | bool | `false` | Follow log output as it is written (tail -f) |
| `--full` | — | bool | `false` | Show the full log (shortcut for --tail -1) |
| `--no-tui` | — | bool | `false` | Force TUI off (escape hatch if TUI ever becomes default) |
| `--raw` | — | bool | `false` | For agentic sessions, print the raw transcript JSONL verbatim instead of the rendered replay |
| `--strip-ansi` | — | bool | `false` | Strip ANSI color codes from output |
| `--tail` | `-n` | int | `200` | Show the last N lines (0 = none, -1 = all) |
| `--tui` | — | bool | `false` | Open interactive TUI (arrow keys to navigate, enter to view details, c to copy id) |

## vigolium log ls

List native scan and agentic scan sessions

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--no-tui` | — | bool | `false` | Force TUI off (escape hatch if TUI ever becomes default) |
| `--tui` | — | bool | `false` | Open interactive TUI (arrow keys to navigate, enter to view details, c to copy id) |

## vigolium module

Manage scanner modules

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--list-enabled` | — | bool | `false` | Show only enabled modules |
| `--tags` | — | bool | `false` | Show only unique module tags |
| `--type` | — | string | `all` | Filter modules by type: all, active, or passive |

## vigolium module disable

Disable modules matching a search term

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--id` | — | bool | `false` | Match exact module ID instead of fuzzy search |

## vigolium module enable

Enable modules matching a search term

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--id` | — | bool | `false` | Match exact module ID instead of fuzzy search |

## vigolium module ls

List available modules

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--list-enabled` | — | bool | `false` | Show only enabled modules |
| `--tags` | — | bool | `false` | Show only unique module tags |
| `--type` | — | string | `all` | Filter modules by type: all, active, or passive |

## vigolium olium

Launch vigolium — interactive TUI agent (alias for `vigolium agent olium`)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--base-url` | — | string | — | Endpoint URL for openai-compatible provider (e.g. http://localhost:11434/v1 for Ollama); falls back to agent.olium.custom_provider.base_url |
| `--bridge-bin` | — | string | — | Path to the 'vigolium-audit' binary hosting the SDK bridge (anthropic-claude-sdk-bridge provider; default: embedded blob, then PATH) |
| `--claude-bin` | — | string | — | Path to the 'claude' binary (anthropic-cli provider) |
| `--gcp-location` | — | string | — | GCP region for Vertex providers (else $GOOGLE_CLOUD_LOCATION, then YAML, then us-central1) |
| `--gcp-project` | — | string | — | GCP project for Vertex providers (else $GOOGLE_CLOUD_PROJECT, then YAML, then SA file's project_id) |
| `--llm-api-key` | — | string | — | API key for key-based providers (anthropic-api-key, openai-api-key); else uses ANTHROPIC_API_KEY / OPENAI_API_KEY env |
| `--model` | — | string | — | Model id (provider-specific default if empty) |
| `--oauth-cred` | — | string | — | Path to OAuth/SA credential file (openai-codex-oauth: ~/.codex/auth.json; anthropic-vertex/google-vertex: SA JSON or $GOOGLE_APPLICATION_CREDENTIALS) |
| `--oauth-token` | — | string | — | Anthropic OAuth bearer token (anthropic-oauth; falls back to agent.olium.oauth_token or $ANTHROPIC_API_KEY) |
| `--prompt` | `-p` | string | — | Run one prompt non-interactively and stream to stdout (skips the TUI). Pass '-' to read the prompt from stdin |
| `--provider` | — | string | — | Olium provider override: openai-codex-oauth \| openai-api-key \| openai-responses \| anthropic-api-key \| anthropic-oauth \| anthropic-cli \| anthropic-claude-sdk-bridge \| anthropic-vertex \| google-vertex \| openai-compatible \| anthropic-compatible (falls back to agent.olium.provider config); default openai-compatible |
| `--stdin` | — | bool | `false` | Force reading prompt from stdin |
| `--system` | — | string | — | Override system prompt |

## vigolium project create

Create a new project

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--description` | — | string | — | Project description |

## vigolium project delete

Delete a project and every record tied to it

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--keep-config` | — | bool | `false` | Keep the project's config directory (~/.vigolium/projects/<uuid>) on disk |

## vigolium project list

List all projects

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--no-tui` | — | bool | `false` | Force TUI off (escape hatch if TUI ever becomes default) |
| `--tui` | — | bool | `false` | Open interactive TUI (arrow keys to navigate, enter to view details, c to copy id) |

## vigolium project use

Print the shell export command to set the active project

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--description` | — | string | — | Description to use when auto-creating a project |
| `--name` | — | string | — | Name to use when auto-creating a project (default: "Project <short-uuid>") |

## vigolium replay

Re-send a stored or supplied HTTP request and diff baseline vs replay

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--all` | `-a` | bool | `false` | Bulk: replay every matched stored record (lifts the -n/--limit cap); re-send all stored traffic |
| `--asc` | — | bool | `false` | Bulk: sort ascending (default: descending) |
| `--auth-session` | — | string | — | Auth session name to merge headers from (from 'vigolium auth list') |
| `--body` | — | string | — | Bulk: filter records whose request/response body contains this text |
| `--burp-bridge-url` | `-B` | string | — | Loopback Burp/Caido bridge URL used by --save-to-burp / --send-via-burp / --to-repeater / --to-organizer (alias: --caido-bridge-url) |
| `--concurrency` | `-c` | int | `10` | Bulk: concurrent replays; keep low to avoid overwhelming an intercepting proxy like Burp |
| `--exclude-body` | — | string | — | Bulk: drop records whose request/response body contains the term (inverse of --body) |
| `--exclude-header-search` | — | string | — | Bulk: drop records whose HTTP header names/values contain the term (inverse of --header-search) |
| `--exclude-search` | — | stringArray | — | Bulk: drop records where the term appears in the URL, path, or raw request/response (repeatable; dropped if ANY term matches — inverse of --search) |
| `--finding-id` | — | int64 | `0` | Finding ID — replay the finding's linked record (or its stored evidence) |
| `--from` | — | string | — | Bulk: only records at or after this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339 (alias: --since) |
| `--header` | `-H` | stringArray | — | Extra request header 'Name: value' (repeatable, overrides baseline) |
| `--header-search` | — | string | — | Bulk: filter records by text in HTTP header names/values (the search filter `traffic --header` applies; -H/--header overrides headers instead) |
| `--highlight` | — | string | — | Highlight colour for the --to-organizer item: none\|red\|orange\|yellow\|green\|cyan\|blue\|pink\|magenta\|gray |
| `--host` | — | string | — | Bulk: filter records by hostname pattern (wildcard supported) |
| `--http-mode` | — | string | — | With --send-via-burp: wire protocol — auto\|http1\|http2\|http2_ignore_alpn (default auto; use http1 for request smuggling/desync) |
| `--in-replace` | — | bool | `false` | When the source is a stored record, update its stored response with the replay |
| `--input` | `-i` | string | — | Raw input: curl, raw HTTP, Burp XML, base64, URL, or '-' for stdin |
| `--input-file` | — | string | — | Read --input value from a file |
| `--limit` | `-n` | int | `100` | Bulk: max records to replay (use --all to lift the cap) |
| `--method` | — | stringSlice | — | Bulk: filter records by HTTP method (repeatable) |
| `--no-cookies` | — | bool | `false` | Don't carry cookies (overrides --session-id) |
| `--no-redirects` | — | bool | `false` | Don't follow 30x redirects |
| `--notes` | — | string | — | Note attached to the --to-organizer item (<=200 chars) |
| `--offset` | — | int | `0` | Bulk: skip this many matched records before replaying (pagination) |
| `--output` | `-o` | string | — | Write JSON result to this file (default: stdout) |
| `--path` | — | string | — | Bulk: filter records by URL path pattern |
| `--pretty` | — | bool | `false` | Human-readable summary instead of JSON |
| `--raw-request` | — | string | — | Full raw HTTP request override — send these exact bytes instead of the resolved baseline |
| `--raw-request-file` | — | string | — | Read --raw-request from a file |
| `--record-uuid` | `-u` | string | — | Stored HTTP record UUID to use as baseline |
| `--repeater-tab` | — | string | — | Repeater tab name for --to-repeater (default: vigolium) |
| `--save-to-burp` | — | bool | `false` | Add each replayed request and its fresh response to Burp's Target Site map |
| `--search` | — | stringArray | — | Bulk: search across URL, path, and the raw request/response (headers + body); repeatable, AND-combined |
| `--send-timeout` | — | duration | `0s` | With --send-via-burp: response timeout (<=2m; default uses the bridge's 30s) |
| `--send-via-burp` | — | bool | `false` | Send the request through Burp's own HTTP stack (exact bytes — malformed/smuggling preserved) instead of Go's client; requires --burp-bridge-url |
| `--session-id` | — | string | — | Persist cookies across calls under ~/.vigolium/replay-jars/<id>.json |
| `--sort` | — | string | `created_at` | Bulk: sort matched records by: uuid, created_at, sent_at, method, status, time |
| `--source` | — | string | — | Bulk: filter records by source (burp, caido, scanner, ingest-cli, ingest-proxy, seed, ...) |
| `--stateless` | `-S` | bool | `false` | Read records from --db (a .jsonl export or standalone .sqlite) with project scoping off; never writes to your project DB |
| `--status` | — | intSlice | — | Bulk: filter records by stored status code (repeatable) |
| `--target` | `-t` | string | — | Override scheme/host/port (e.g. https://staging.example.com) |
| `--timeout` | — | duration | `25s` | Per-request timeout (e.g. 30s, 1m) |
| `--to` | — | string | — | Bulk: only records at or before this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339; a bare date covers the whole day (alias: --until) |
| `--to-organizer` | — | bool | `false` | Store the replayed request + response in Burp's Organizer for manual follow-up; requires --burp-bridge-url |
| `--to-repeater` | — | bool | `false` | Stage the replayed request in a Burp Repeater tab for manual testing; requires --burp-bridge-url |
| `--with-browser` | — | bool | `false` | Load each record's URL in a real browser routed via --proxy so an intercepting proxy captures browser-driven traffic; navigation-only, so no baseline-vs-replay diff is produced |

## vigolium run

Run a single native scan phase (alias for scan --only <phase>)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--advanced-options` | `-a` | stringToString | — | Module-specific options as key=value (e.g. -a xss.dom=true) |
| `--auth` | — | stringArray | — | Inline session in 'name:Header:value' format. Repeatable; commas are literal (header values may contain commas). |
| `--auth-file` | — | stringArray | — | Path to auth file (YAML/JSON, single session or sessions: bundle), or bare name resolved against scanning_strategy.session.session_dir. Repeatable; commas are literal. |
| `--browser-engine` | `-E` | string | `chromium` | Browser engine: 'chromium', 'ungoogled', or 'fingerprint' |
| `--browsers` | `-b` | int | `1` | Number of parallel browser instances for spidering |
| `--concurrency` | `-c` | int | `50` | Number of concurrent scan workers |
| `--db-isolate` | — | bool | `false` | Scan into a private temporary database, then merge results into --db (or the default DB) at the end — lets many parallel scans share one --db without write contention (SQLite only, not with --stateless; combine with -P -T to fan out targets and export one unified output from the merged DB) |
| `--discover` | — | bool | `false` | Enable content discovery phase before scanning |
| `--discover-max-time` | — | duration | `1h0m0s` | Max time for content discovery per target |
| `--external-harvest` | — | bool | `false` | Enable external intelligence gathering phase (Wayback, CT logs, etc.) |
| `--fail-on` | — | string | — | Exit non-zero if a finding at or above this severity is present (info\|low\|medium\|high\|critical) — for CI/agent gating. Scoped to this scan; --soft-fail overrides; with -P it is evaluated per child. |
| `--follow-subdomains` | — | bool | `false` | Pull in-scope subdomains discovered in responses into the scan (exact hosts only, not the whole apex; auto-on at --intensity deep) |
| `--fuzz-wordlist` | — | string | — | Custom fuzz wordlist path for discovery (enables fuzzing on the fly) |
| `--headed` | — | bool | `false` | Show the browser window during spidering (sugar for --headless=false; wins when both are set) |
| `--header` | `-H` | stringArray | — | Add custom HTTP header (repeatable, e.g. -H 'Auth: Bearer token'). Commas are literal — repeat -H for multiple headers. |
| `--headless` | — | bool | `true` | Run browser in headless mode |
| `--heuristics-check` | — | string | — | Pre-scan heuristics level: none, basic, advanced (default: basic) |
| `--include-response` | — | bool | `false` | Include full HTTP response body in output |
| `--input` | `-i` | string | `-` | Input file path or spec (use - for stdin) |
| `--input-mode` | `-I` | string | `urls` | Input format: urls, openapi, swagger, wsdl, burp, curl, nuclei, har (see --list-input-mode) |
| `--input-read-timeout` | — | duration | `3m0s` | Timeout for reading input from stdin or file |
| `--intensity` | — | string | — | Scan intensity preset: quick, balanced, or deep (maps to scanning profile + strategy) |
| `--known-issue-scan-exclude-tags` | — | stringSlice | — | Nuclei template tags to exclude (comma-separated) |
| `--known-issue-scan-severities` | — | stringSlice | — | Filter Nuclei templates by severity (critical,high,medium,low,info) |
| `--known-issue-scan-tags` | — | stringSlice | — | Nuclei template tags to include (comma-separated) |
| `--known-issue-scan-templates-dir` | — | string | — | Custom Nuclei templates directory |
| `--max-findings-per-module` | — | int | `10` | Stop reporting after N findings per module (0 = unlimited) |
| `--max-host-error` | — | int | `30` | Skip host after reaching this many consecutive errors |
| `--max-per-host` | — | int | `50` | Maximum concurrent requests allowed per host |
| `--module-tag` | — | stringSlice | — | Filter modules by tag (OR condition, e.g. --module-tag spring --module-tag injection) |
| `--modules` | `-m` | stringSlice | — | Scan modules to enable (default "all", supports fuzzy match on ID/name, e.g. -m xss -m sqli) |
| `--no-carry-browser-session` | — | bool | `false` | Do not carry the spidering browser's cleared session (cookies + UA) into discovery/scanning (on by default when --spider runs; scoped to the same host, respects -H) |
| `--no-cdp` | — | bool | `false` | Disable Chrome DevTools Protocol event listener detection |
| `--no-clustering` | — | bool | `false` | Disable deduplication of identical concurrent HTTP requests |
| `--no-forms` | — | bool | `false` | Disable automatic form detection and filling during spidering |
| `--no-prefix-breaker` | — | bool | `false` | Disable per-prefix circuit breaker that stops discovery from recursing into trap directories |
| `--no-tech-filter` | — | bool | `false` | Disable the tech-stack allowlist (run every module regardless of detected stack). Auto-enabled by --intensity=deep. |
| `--no-waf-pacing` | — | bool | `false` | Disable proactive CDN/WAF-edge pacing (don't pre-throttle per-host concurrency when a CloudFront/Cloudflare/etc. edge is detected); reactive back-off after a WAF block still applies |
| `--oast-url` | — | string | — | Fixed out-of-band callback URL (overrides auto-generated interactsh URL) |
| `--omit-response` | — | bool | `false` | Omit raw HTTP request/response bytes from output file (keeps metadata, smaller files) |
| `--only` | — | string | — | Run only these phases (comma-separated: ingestion, discovery, external-harvest, spidering, known-issue-scan, dynamic-assessment, extension) |
| `--output` | `-o` | string | — | Write findings to specified output file |
| `--parallel` | `-P` | int | `1` | Scan up to N targets concurrently as isolated child processes (requires -S -T --split-by-host, OR --db-isolate -T which merges into --db and exports one unified output; each target keeps its own --concurrency, so real in-flight requests ≈ N × --concurrency) |
| `--port-sweep-ports` | — | string | — | Override the alternate HTTP(S) ports swept on CLI target hosts (comma-separated; sweep runs at --intensity deep or --follow-subdomains) |
| `--print-finding` | — | bool | `false` | After the scan, print each finding to stdout as Markdown (description + matched evidence + request/response), like 'vigolium finding --markdown'. Pairs well with -S and --silent for a quick scan. |
| `--print-traffic` | — | bool | `false` | After the scan, print the run's raw HTTP request/response pairs to stdout, like 'vigolium traffic --raw'. Pairs well with -S and --silent. |
| `--print-traffic-tree` | — | bool | `false` | After the scan, print the run's HTTP traffic to stdout as a host/path hierarchy tree, like 'vigolium traffic --tree'. Pairs well with -S and --silent. |
| `--rate-limit` | `-r` | int | `100` | Global requests/second cap, enforced across native scanning and known-issue-scan when set (unset = per-host concurrency only) |
| `--report-url` | — | string | — | URL for the "Raw Report URL" button in HTML reports (overrides VIGOLIUM_REPORT_SHARED_URL) |
| `--required-only` | — | bool | `false` | Parse only required fields from input format (ignore optional) |
| `--resume` | — | bool | `false` | Resume a prior -S -T --split-by-host -P run from its progress manifest (<output>.progress.json): skip targets that already completed cleanly and scan only the remainder. Run bare ('vigolium scan --resume', no other flags) to auto-discover the *.progress.json in the current directory and relaunch the saved run from it (pass -o <prefix> to disambiguate when several exist) |
| `--retries` | — | int | `1` | Number of retry attempts for failed requests |
| `--scanning-max-duration` | — | duration | `0s` | Maximum total scan duration (overrides config, e.g. 1h, 30m) |
| `--scanning-profile` | — | string | — | Scanning profile name or YAML file path |
| `--scope-origin` | — | string | — | Host scope strictness: all, relaxed, balanced, strict |
| `--skip` | — | stringSlice | — | Skip these phases (repeatable: discovery, external-harvest, spidering, known-issue-scan, dynamic-assessment) |
| `--skip-format-validation` | — | bool | `false` | Skip validation of input file format |
| `--skip-heuristics` | — | bool | `false` | Disable pre-scan heuristics (equivalent to --heuristics-check=none) |
| `--spec-default` | — | string | `1` | Fallback value for required OpenAPI parameters that lack examples |
| `--spec-header` | — | stringArray | — | Add HTTP header to OpenAPI-generated requests (repeatable; commas are literal) |
| `--spec-url` | — | bool | `false` | Use base URLs from the OpenAPI spec's servers field |
| `--spec-var` | — | stringSlice | — | Set OpenAPI parameter value as key=value (repeatable) |
| `--spider` | — | bool | `false` | Enable browser-based spidering phase before scanning |
| `--spider-max-time` | — | duration | `30m0s` | Max time for spidering per target |
| `--split-by-host` | — | bool | `false` | In stateless multi-target mode (-S -T file), write a separate per-host output file (base-<host>.<ext>) instead of one unified file |
| `--stateless` | `-S` | bool | `false` | Use a temporary database that is discarded after the scan (pass --output/--format to persist results) |
| `--stats` | — | bool | `false` | Show live progress stats during scanning |
| `--strategy` | — | string | — | Scanning strategy preset (lite, balanced, deep) |
| `--stream` | — | bool | `false` | Process targets as a stream without buffering or deduplication |
| `--target` | `-t` | stringArray | — | Target URL to scan (repeatable). Commas are literal so a URL query like ?ids=1,2,3 stays one target — repeat -t for multiple targets. |
| `--target-file` | `-T` | stringArray | — | File containing target URLs (one per line; repeatable for multiple files). Commas in the path are literal. |
| `--timeout` | — | duration | `15s` | HTTP request timeout (e.g. 30s, 1m, 2h) |
| `--upload-results` | — | bool | `false` | Upload scan results to cloud storage after completion (requires storage config) |

## vigolium scan

Run a native scan — deterministic multi-phase vulnerability scanning

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--advanced-options` | `-a` | stringToString | — | Module-specific options as key=value (e.g. -a xss.dom=true) |
| `--auth` | — | stringArray | — | Inline session in 'name:Header:value' format. Repeatable; commas are literal (header values may contain commas). |
| `--auth-file` | — | stringArray | — | Path to auth file (YAML/JSON, single session or sessions: bundle), or bare name resolved against scanning_strategy.session.session_dir. Repeatable; commas are literal. |
| `--browser-engine` | `-E` | string | `chromium` | Browser engine: 'chromium', 'ungoogled', or 'fingerprint' |
| `--browsers` | `-b` | int | `1` | Number of parallel browser instances for spidering |
| `--concurrency` | `-c` | int | `50` | Number of concurrent scan workers |
| `--db-isolate` | — | bool | `false` | Scan into a private temporary database, then merge results into --db (or the default DB) at the end — lets many parallel scans share one --db without write contention (SQLite only, not with --stateless; combine with -P -T to fan out targets and export one unified output from the merged DB) |
| `--discover` | — | bool | `false` | Enable content discovery phase before scanning |
| `--discover-max-time` | — | duration | `1h0m0s` | Max time for content discovery per target |
| `--external-harvest` | — | bool | `false` | Enable external intelligence gathering phase (Wayback, CT logs, etc.) |
| `--fail-on` | — | string | — | Exit non-zero if a finding at or above this severity is present (info\|low\|medium\|high\|critical) — for CI/agent gating. Scoped to this scan; --soft-fail overrides; with -P it is evaluated per child. |
| `--follow-subdomains` | — | bool | `false` | Pull in-scope subdomains discovered in responses into the scan (exact hosts only, not the whole apex; auto-on at --intensity deep) |
| `--fuzz-wordlist` | — | string | — | Custom fuzz wordlist path for discovery (enables fuzzing on the fly) |
| `--headed` | — | bool | `false` | Show the browser window during spidering (sugar for --headless=false; wins when both are set) |
| `--header` | `-H` | stringArray | — | Add custom HTTP header (repeatable, e.g. -H 'Auth: Bearer token'). Commas are literal — repeat -H for multiple headers. |
| `--headless` | — | bool | `true` | Run browser in headless mode |
| `--heuristics-check` | — | string | — | Pre-scan heuristics level: none, basic, advanced (default: basic) |
| `--include-response` | — | bool | `false` | Include full HTTP response body in output |
| `--input` | `-i` | string | `-` | Input file path or spec (use - for stdin) |
| `--input-mode` | `-I` | string | `urls` | Input format: urls, openapi, swagger, wsdl, burp, curl, nuclei, har (see --list-input-mode) |
| `--input-read-timeout` | — | duration | `3m0s` | Timeout for reading input from stdin or file |
| `--intensity` | — | string | — | Scan intensity preset: quick, balanced, or deep (maps to scanning profile + strategy) |
| `--known-issue-scan-exclude-tags` | — | stringSlice | — | Nuclei template tags to exclude (comma-separated) |
| `--known-issue-scan-severities` | — | stringSlice | — | Filter Nuclei templates by severity (critical,high,medium,low,info) |
| `--known-issue-scan-tags` | — | stringSlice | — | Nuclei template tags to include (comma-separated) |
| `--known-issue-scan-templates-dir` | — | string | — | Custom Nuclei templates directory |
| `--max-findings-per-module` | — | int | `10` | Stop reporting after N findings per module (0 = unlimited) |
| `--max-host-error` | — | int | `30` | Skip host after reaching this many consecutive errors |
| `--max-per-host` | — | int | `50` | Maximum concurrent requests allowed per host |
| `--module-id` | — | stringSlice | — | Run exactly these module IDs (exact match against active AND passive modules, repeatable). Unlike -m, also selects passive modules. |
| `--module-tag` | — | stringSlice | — | Filter modules by tag (OR condition, e.g. --module-tag spring --module-tag injection) |
| `--modules` | `-m` | stringSlice | — | Scan modules to enable (default "all", supports fuzzy match on ID/name, e.g. -m xss -m sqli) |
| `--no-carry-browser-session` | — | bool | `false` | Do not carry the spidering browser's cleared session (cookies + UA) into discovery/scanning (on by default when --spider runs; scoped to the same host, respects -H) |
| `--no-cdp` | — | bool | `false` | Disable Chrome DevTools Protocol event listener detection |
| `--no-clustering` | — | bool | `false` | Disable deduplication of identical concurrent HTTP requests |
| `--no-forms` | — | bool | `false` | Disable automatic form detection and filling during spidering |
| `--no-prefix-breaker` | — | bool | `false` | Disable per-prefix circuit breaker that stops discovery from recursing into trap directories |
| `--no-tech-filter` | — | bool | `false` | Disable the tech-stack allowlist (run every module regardless of detected stack). Auto-enabled by --intensity=deep. |
| `--no-waf-pacing` | — | bool | `false` | Disable proactive CDN/WAF-edge pacing (don't pre-throttle per-host concurrency when a CloudFront/Cloudflare/etc. edge is detected); reactive back-off after a WAF block still applies |
| `--oast-url` | — | string | — | Fixed out-of-band callback URL (overrides auto-generated interactsh URL) |
| `--omit-response` | — | bool | `false` | Omit raw HTTP request/response bytes from output file (keeps metadata, smaller files) |
| `--only` | — | string | — | Run only these phases (comma-separated: ingestion, discovery, external-harvest, spidering, known-issue-scan, dynamic-assessment, extension) |
| `--output` | `-o` | string | — | Write findings to specified output file |
| `--parallel` | `-P` | int | `1` | Scan up to N targets concurrently as isolated child processes (requires -S -T --split-by-host, OR --db-isolate -T which merges into --db and exports one unified output; each target keeps its own --concurrency, so real in-flight requests ≈ N × --concurrency) |
| `--passive-only` | — | bool | `false` | Run only passive modules (no active scanning). Combine with --module-id to narrow to specific passive modules. |
| `--port-sweep-ports` | — | string | — | Override the alternate HTTP(S) ports swept on CLI target hosts (comma-separated; sweep runs at --intensity deep or --follow-subdomains) |
| `--print-finding` | — | bool | `false` | After the scan, print each finding to stdout as Markdown (description + matched evidence + request/response), like 'vigolium finding --markdown'. Pairs well with -S and --silent for a quick scan. |
| `--print-traffic` | — | bool | `false` | After the scan, print the run's raw HTTP request/response pairs to stdout, like 'vigolium traffic --raw'. Pairs well with -S and --silent. |
| `--print-traffic-tree` | — | bool | `false` | After the scan, print the run's HTTP traffic to stdout as a host/path hierarchy tree, like 'vigolium traffic --tree'. Pairs well with -S and --silent. |
| `--rate-limit` | `-r` | int | `100` | Global requests/second cap, enforced across native scanning and known-issue-scan when set (unset = per-host concurrency only) |
| `--report-url` | — | string | — | URL for the "Raw Report URL" button in HTML reports (overrides VIGOLIUM_REPORT_SHARED_URL) |
| `--required-only` | — | bool | `false` | Parse only required fields from input format (ignore optional) |
| `--resume` | — | bool | `false` | Resume a prior -S -T --split-by-host -P run from its progress manifest (<output>.progress.json): skip targets that already completed cleanly and scan only the remainder. Run bare ('vigolium scan --resume', no other flags) to auto-discover the *.progress.json in the current directory and relaunch the saved run from it (pass -o <prefix> to disambiguate when several exist) |
| `--retries` | — | int | `1` | Number of retry attempts for failed requests |
| `--scanning-max-duration` | — | duration | `0s` | Maximum total scan duration (overrides config, e.g. 1h, 30m) |
| `--scanning-profile` | — | string | — | Scanning profile name or YAML file path |
| `--scope-origin` | — | string | — | Host scope strictness: all, relaxed, balanced, strict |
| `--skip` | — | stringSlice | — | Skip these phases (repeatable: discovery, external-harvest, spidering, known-issue-scan, dynamic-assessment) |
| `--skip-format-validation` | — | bool | `false` | Skip validation of input file format |
| `--skip-heuristics` | — | bool | `false` | Disable pre-scan heuristics (equivalent to --heuristics-check=none) |
| `--spec-default` | — | string | `1` | Fallback value for required OpenAPI parameters that lack examples |
| `--spec-header` | — | stringArray | — | Add HTTP header to OpenAPI-generated requests (repeatable; commas are literal) |
| `--spec-url` | — | bool | `false` | Use base URLs from the OpenAPI spec's servers field |
| `--spec-var` | — | stringSlice | — | Set OpenAPI parameter value as key=value (repeatable) |
| `--spider` | — | bool | `false` | Enable browser-based spidering phase before scanning |
| `--spider-max-time` | — | duration | `30m0s` | Max time for spidering per target |
| `--split-by-host` | — | bool | `false` | In stateless multi-target mode (-S -T file), write a separate per-host output file (base-<host>.<ext>) instead of one unified file |
| `--stateless` | `-S` | bool | `false` | Use a temporary database that is discarded after the scan (pass --output/--format to persist results) |
| `--stats` | — | bool | `false` | Show live progress stats during scanning |
| `--strategy` | — | string | — | Scanning strategy preset (lite, balanced, deep) |
| `--stream` | — | bool | `false` | Process targets as a stream without buffering or deduplication |
| `--target` | `-t` | stringArray | — | Target URL to scan (repeatable). Commas are literal so a URL query like ?ids=1,2,3 stays one target — repeat -t for multiple targets. |
| `--target-file` | `-T` | stringArray | — | File containing target URLs (one per line; repeatable for multiple files). Commas in the path are literal. |
| `--timeout` | — | duration | `15s` | HTTP request timeout (e.g. 30s, 1m, 2h) |
| `--upload-results` | — | bool | `false` | Upload scan results to cloud storage after completion (requires storage config) |

## vigolium scan-request

Scan a raw HTTP request for vulnerabilities

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--concurrency` | `-c` | int | `50` | Number of concurrent scan workers |
| `--discover` | — | bool | `false` | Run content discovery before scanning |
| `--external-harvest` | — | bool | `false` | Run external intelligence harvesting before scanning |
| `--fail-on` | — | string | — | Exit non-zero if a finding at or above this severity is present (info\|low\|medium\|high\|critical) — for CI/agent gating; --soft-fail overrides. |
| `--input` | `-i` | string | `-` | Input file or - for stdin |
| `--known-issue-scan` | — | bool | `false` | Run known issue scan (Nuclei + native secret scanning) |
| `--max-findings-per-module` | — | int | `10` | Stop reporting after N findings per module (0 = unlimited) |
| `--max-host-error` | — | int | `30` | Skip host after reaching this many consecutive errors |
| `--max-per-host` | — | int | `50` | Maximum concurrent requests allowed per host |
| `--module-id` | — | stringSlice | — | Run exactly these module IDs (exact match against active AND passive modules, repeatable). Unlike -m, also selects passive modules. |
| `--module-tag` | — | stringSlice | — | Filter modules by tag (OR condition, e.g. --module-tag spring --module-tag injection) |
| `--modules` | `-m` | stringSlice | — | Scan modules to enable (default "all", supports fuzzy match on ID/name, e.g. -m xss -m sqli) |
| `--no-clustering` | — | bool | `false` | Disable deduplication of identical concurrent HTTP requests |
| `--no-passive` | — | bool | `false` | Skip passive modules |
| `--no-tech-filter` | — | bool | `false` | Disable the tech-stack allowlist (run every module regardless of detected stack). Auto-enabled by --intensity=deep. |
| `--no-waf-pacing` | — | bool | `false` | Disable proactive CDN/WAF-edge pacing (don't pre-throttle per-host concurrency when a CloudFront/Cloudflare/etc. edge is detected); reactive back-off after a WAF block still applies |
| `--output` | `-o` | string | — | Write findings to this file (use with --format jsonl\|html; pairs with -S/--stateless) |
| `--passive-only` | — | bool | `false` | Run only passive modules (no active scanning). Combine with --module-id to narrow to specific passive modules. |
| `--print-finding` | — | bool | `false` | After the scan, print each finding to stdout as Markdown (description + matched evidence + request/response), like 'vigolium finding --markdown'. Pairs well with -S and --silent for a quick single-target scan. |
| `--print-traffic` | — | bool | `false` | After the scan, print the run's raw HTTP request/response pairs to stdout, like 'vigolium traffic --raw'. Pairs well with -S and --silent. |
| `--print-traffic-tree` | — | bool | `false` | After the scan, print the run's HTTP traffic to stdout as a host/path hierarchy tree, like 'vigolium traffic --tree'. Pairs well with -S and --silent. |
| `--rate-limit` | `-r` | int | `100` | Global requests/second cap, enforced across native scanning and known-issue-scan when set (unset = per-host concurrency only) |
| `--skip` | — | stringSlice | — | Skip these phases (repeatable: discovery, external-harvest, spidering, known-issue-scan, dynamic-assessment) |
| `--spider` | — | bool | `false` | Run browser-based spidering before scanning |
| `--stateless` | `-S` | bool | `false` | Use a temporary database that is discarded after the scan (pass --output/--format to persist results) |
| `--target` | `-t` | string | — | Override target URL (scheme://host) |
| `--timeout` | — | duration | `15s` | HTTP request timeout (e.g. 30s, 1m, 2h) |

## vigolium scan-url

Scan a single URL for vulnerabilities

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--body` | — | string | — | Request body |
| `--concurrency` | `-c` | int | `50` | Number of concurrent scan workers |
| `--discover` | — | bool | `false` | Run content discovery before scanning |
| `--external-harvest` | — | bool | `false` | Run external intelligence harvesting before scanning |
| `--fail-on` | — | string | — | Exit non-zero if a finding at or above this severity is present (info\|low\|medium\|high\|critical) — for CI/agent gating; --soft-fail overrides. |
| `--header` | `-H` | stringArray | — | Custom header (repeatable, e.g. -H 'Cookie: x=1'). Commas are literal — repeat -H for multiple headers. |
| `--known-issue-scan` | — | bool | `false` | Run known issue scan (Nuclei + native secret scanning) |
| `--max-findings-per-module` | — | int | `10` | Stop reporting after N findings per module (0 = unlimited) |
| `--max-host-error` | — | int | `30` | Skip host after reaching this many consecutive errors |
| `--max-per-host` | — | int | `50` | Maximum concurrent requests allowed per host |
| `--method` | — | string | `GET` | HTTP method |
| `--module-id` | — | stringSlice | — | Run exactly these module IDs (exact match against active AND passive modules, repeatable). Unlike -m, also selects passive modules. |
| `--module-tag` | — | stringSlice | — | Filter modules by tag (OR condition, e.g. --module-tag spring --module-tag injection) |
| `--modules` | `-m` | stringSlice | — | Scan modules to enable (default "all", supports fuzzy match on ID/name, e.g. -m xss -m sqli) |
| `--no-clustering` | — | bool | `false` | Disable deduplication of identical concurrent HTTP requests |
| `--no-passive` | — | bool | `false` | Skip passive modules |
| `--no-tech-filter` | — | bool | `false` | Disable the tech-stack allowlist (run every module regardless of detected stack). Auto-enabled by --intensity=deep. |
| `--no-waf-pacing` | — | bool | `false` | Disable proactive CDN/WAF-edge pacing (don't pre-throttle per-host concurrency when a CloudFront/Cloudflare/etc. edge is detected); reactive back-off after a WAF block still applies |
| `--output` | `-o` | string | — | Write findings to this file (use with --format jsonl\|html; pairs with -S/--stateless) |
| `--passive-only` | — | bool | `false` | Run only passive modules (no active scanning). Combine with --module-id to narrow to specific passive modules. |
| `--print-finding` | — | bool | `false` | After the scan, print each finding to stdout as Markdown (description + matched evidence + request/response), like 'vigolium finding --markdown'. Pairs well with -S and --silent for a quick single-target scan. |
| `--print-traffic` | — | bool | `false` | After the scan, print the run's raw HTTP request/response pairs to stdout, like 'vigolium traffic --raw'. Pairs well with -S and --silent. |
| `--print-traffic-tree` | — | bool | `false` | After the scan, print the run's HTTP traffic to stdout as a host/path hierarchy tree, like 'vigolium traffic --tree'. Pairs well with -S and --silent. |
| `--rate-limit` | `-r` | int | `100` | Global requests/second cap, enforced across native scanning and known-issue-scan when set (unset = per-host concurrency only) |
| `--skip` | — | stringSlice | — | Skip these phases (repeatable: discovery, external-harvest, spidering, known-issue-scan, dynamic-assessment) |
| `--spider` | — | bool | `false` | Run browser-based spidering before scanning |
| `--stateless` | `-S` | bool | `false` | Use a temporary database that is discarded after the scan (pass --output/--format to persist results) |
| `--target` | `-t` | stringArray | — | Target URL to scan (repeatable; alternative to the positional URL argument). Commas are literal. |
| `--timeout` | — | duration | `15s` | HTTP request timeout (e.g. 30s, 1m, 2h) |

## vigolium server

Start API server

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--alternative-ingest-key` | — | stringSlice | — | Additional API key for ingestion endpoints (repeatable) |
| `--burp-bridge-url` | `-B` | string | — | Merge live proxy traffic from this loopback Burp/Caido bridge URL into /api/http-records (alias: --caido-bridge-url) |
| `--catchup-threads` | — | int | `4` | Deprecated: no-op (catch-up scanning is disabled) |
| `--demo-only` | — | bool | `false` | Expose only the demo allowlist: GET /api/findings[/:id], /api/http-records[/:uuid], /api/modules, /api/stats, /api/extensions[/:name\|/docs] |
| `--disable-catchup` | — | bool | `false` | Deprecated: no-op (catch-up scanning is already disabled) |
| `--disable-warm-session` | — | bool | `false` | Disable agent subprocess warm session pooling |
| `--export-ca` | — | string | — | Write the ingest-proxy MITM CA certificate to this path and exit (generates the CA if needed) |
| `--full-native-scan-on-receive` | — | bool | `false` | Run the full native scan pipeline (discovery + spidering + dynamic-assessment) continuously on received records, instead of dynamic-assessment only |
| `--host` | — | string | `0.0.0.0` | Bind address for the API server |
| `--ingest-proxy-port` | — | int | `0` | Transparent HTTP proxy port for recording traffic (0 = disabled) |
| `--mem-buffer` | — | int | `10000` | In-memory queue capacity before spilling to disk |
| `--mirror-fs` | — | string | — | Mirror ingested traffic + findings to a live flat filesystem tree under this dir (<dir>/traffic, <dir>/findings), in addition to the database — readable by an external agent with ls/grep/jq |
| `--no-agent` | — | bool | `false` | Disable all agent endpoints and warm session pooling |
| `--no-auth` | `-A` | bool | `false` | Run server without API key authentication |
| `--no-swagger` | — | bool | `false` | Disable Swagger UI and API spec endpoint |
| `--output` | `-o` | string | — | Write findings to specified output file |
| `--passive-only` | — | bool | `false` | With -S/--scan-on-receive, run passive modules only (no active scan traffic; includes secret detection) |
| `--proxy-insecure` | — | bool | `false` | When intercepting HTTPS (--proxy-mitm), skip verification of the upstream server's TLS certificate |
| `--proxy-mitm` | — | bool | `false` | Intercept HTTPS through --ingest-proxy-port using a generated CA so TLS traffic is recorded (and scanned with -S). Trust the CA printed at startup |
| `--scan-on-receive` | `-S` | bool | `false` | Continuously scan new HTTP records as they arrive in the database |
| `--service-port` | — | int | `9002` | Port for the REST API server |
| `--timeout` | — | duration | `15s` | HTTP request timeout for background scan workers (e.g. 30s, 1m) |
| `--view-only` | — | bool | `false` | Run server in read-only mode (disables scanning, ingestion, agent, and all write endpoints) |

## vigolium skills get

Print a skill's full content

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--all` | — | bool | `false` | Output every bundled skill |
| `--full` | — | bool | `false` | Include reference files, not just SKILL.md |

## vigolium skills install

Install skill bundle(s) into a coding agent's skills directory

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--agent` | — | string | `claude` | Target coding agent: claude, codex, or agents |
| `--all` | — | bool | `false` | Install every bundled skill |
| `--dir` | — | string | — | Override the destination directory (skips --agent/--scope resolution) |
| `--scope` | — | string | `project` | Install scope: project (current folder) or global (home dir) |

## vigolium storage download

Download an object from cloud storage

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--output` | `-o` | string | — | Write to this file instead of stdout |

## vigolium storage ls

List objects in cloud storage for the active project

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--prefix` | — | string | — | Limit results to keys under this prefix |
| `--tree` | — | bool | `false` | Render objects as a directory tree |

## vigolium storage presign

Generate a presigned upload or download URL

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--expiry` | — | duration | `1h0m0s` | URL validity duration (e.g. 30m, 1h, 24h) |
| `--key` | — | string | — | Object key (required) |
| `--method` | — | string | `GET` | HTTP method: GET or PUT |

## vigolium storage results

Download the result bundle for a scan

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--output` | `-o` | string | — | Write to this file (default: results-<uuid>.tar.gz in cwd) |

## vigolium storage upload

Upload a file to cloud storage

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--content-type` | — | string | — | Content-Type to set on the object |
| `--key` | — | string | — | Object key (default: ugc/<basename>) |

## vigolium traffic

Browse or replay HTTP traffic (alias: db ls --table http_records)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--all` | `-a` | bool | `false` | List/replay every matched record (ignore the -n/--limit cap); pair with --replay to re-send all stored traffic |
| `--asc` | — | bool | `false` | Sort in ascending order (default: descending) |
| `--body` | — | string | — | Search within HTTP request/response body content |
| `--burp` | — | bool | `false` | Display in Burp Suite-style format (colored request/response) |
| `--burp-bridge-url` | `-B` | string | — | Merge live traffic from this loopback Burp/Caido bridge URL with local database records (alias: --caido-bridge-url) |
| `--columns` | — | stringSlice | — | Columns to show (comma-separated, e.g. HOST,METHOD,PATH,STATUS) |
| `--compact` | — | bool | `false` | With --json, emit metadata only (omit request/response bodies). --markdown already compacts response bodies by default; use --full-body to render them whole |
| `--concurrency` | `-c` | int | `10` | Concurrent replays (--replay); keep low to avoid overwhelming an intercepting proxy like Burp |
| `--exclude-body` | — | string | — | Exclude records whose request/response body contains the term (inverse of --body) |
| `--exclude-columns` | — | stringSlice | — | Columns to hide (comma-separated) |
| `--exclude-header` | — | string | — | Exclude records whose HTTP header names/values contain the term (inverse of --header) |
| `--exclude-search` | — | stringArray | — | Exclude records where the term appears in the URL, path, or raw request/response (repeatable; dropped if ANY term matches — the inverse of --search) |
| `--fields` | — | stringSlice | — | Restrict --json output to these top-level keys (comma-separated, e.g. id,severity,url) |
| `--from` | — | string | — | Show records at or after this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339 (alias: --since) |
| `--full-body` | — | bool | `false` | Render complete request/response bodies (no truncation/stubbing) with --json, and whole (uncompacted) bodies with --markdown |
| `--glob-db` | — | string | — | Read across a glob of result files merged into one temporary DB (e.g. --glob-db 'scans/*.sqlite'); implies -S |
| `--header` | — | string | — | Search within HTTP header names and values |
| `--host` | — | string | — | Filter by hostname pattern (wildcard supported) |
| `--in-replace` | — | bool | `false` | With --replay: overwrite each stored response with the new replay response |
| `--limit` | `-n` | int | `100` | Maximum records to display |
| `--markdown` | — | bool | `false` | Render the matched records as Markdown (request/response in fenced http blocks) to stdout; response bodies are compacted to a preview by default (use --full-body for whole bodies) |
| `--method` | — | stringSlice | — | Filter by HTTP method (repeatable, e.g. --method GET --method POST) |
| `--no-tui` | — | bool | `false` | Force TUI off (escape hatch if TUI ever becomes default) |
| `--offset` | — | int | `0` | Number of records to skip (for pagination) |
| `--path` | — | string | — | Filter by URL path pattern |
| `--raw` | — | bool | `false` | Show full raw HTTP request and response |
| `--replay` | — | bool | `false` | Re-send the matched requests and compare original vs new response (instead of listing) |
| `--save-to-burp` | — | bool | `false` | Copy the database records selected by the active filters into Burp's Target Site map |
| `--save-to-vigolium-db` | — | bool | `false` | Persist the live Burp records selected by the active filters into the database |
| `--search` | — | stringArray | — | Search across URL, path, and the raw request/response (headers + body); repeatable, AND-combined (each term further narrows) |
| `--sort` | — | string | `created_at` | Sort by: uuid, created_at, sent_at, method, status, time |
| `--source` | — | string | — | Filter by record source (e.g. burp, caido, scanner, ingest-cli, ingest-server, ingest-proxy, seed) |
| `--stateless` | `-S` | bool | `false` | Read from --db (a .jsonl export or standalone .sqlite) with project scoping off; never writes to your project DB |
| `--status` | — | intSlice | — | Filter by HTTP status code (repeatable, e.g. --status 200 --status 404) |
| `--timeout` | — | duration | `15s` | Per-request timeout for --replay (e.g. 30s, 1m) |
| `--to` | — | string | — | Show records at or before this time — 2d, 12h, 30m, today, yesterday, 2026-08-05, "2026-08-05 14:30", or RFC3339; a bare date covers the whole day (alias: --until) |
| `--tree` | — | bool | `false` | Display as host/path hierarchy tree |
| `--tui` | — | bool | `false` | Open interactive TUI (arrow keys to navigate, enter to view details, c to copy id) |
| `--with-browser` | — | bool | `false` | Replay each URL through a real browser routed via --proxy (--replay), so Burp captures browser-driven traffic |

## vigolium update

Update Vigolium and its nuclei templates to the latest version

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--skip-binary` | — | bool | `false` | Only update nuclei templates; do not reinstall the binary |
| `--skip-templates` | — | bool | `false` | Only reinstall the binary; do not refresh nuclei templates |
