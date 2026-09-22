# Utility toolbox (`vigolium kit`)

> **Related:** [agent-loop.md](agent-loop.md) · [fuzzing.md](fuzzing.md) · [flags.generated.md](flags.generated.md)

`vigolium kit` is a family of small, **stateless** primitives carved out of the
scanner's own engine — no database, no project scope, no scan pipeline. Each is
a one-shot transform or probe built to be shelled out to by an agent: it reads
files/stdin, honours `-j/--json` for machine output, and gates via exit codes.

## Table of Contents

- [Mental model](#mental-model)
- [secret-scan](#secret-scan)
- [js-beautify](#js-beautify)
- [oast (new / poll)](#oast-new--poll)
- [harvest](#harvest)
- [jwt-crack](#jwt-crack)
- [wordlist](#wordlist)
- [payload](#payload)
- [Gotchas](#gotchas)

---

## Mental model

These are **primitives**: they do one job and report raw output, never a
vulnerability verdict. They touch no project DB, so nothing is persisted — pipe
the output where you need it. Every command reads `-` (or no argument) as stdin
and takes `-j/--json` for a single structured object on stdout (human notes go
to stderr, so a pipe stays clean).

| I need to… | Use |
|---|---|
| Scan files/dirs/stdin for leaked credentials | `vigolium kit secret-scan <files\|dirs\|->` |
| Unminify / unpack a JS bundle | `vigolium kit js-beautify <file\|url\|->` |
| Mint OOB callback URLs and drain hits | `vigolium kit oast new` → `vigolium kit oast poll` |
| Collect known URLs for a domain | `vigolium kit harvest <domain…>` |
| Recover a JWT's HMAC secret | `vigolium kit jwt-crack <token>` |
| List / print the built-in wordlists | `vigolium kit wordlist [name]` |
| Emit built-in payloads by class | `vigolium kit payload --class <c>` |

---

## secret-scan

Scan bytes for leaked secrets with the embedded detection catalog (~12k
kingfisher + vigolium rules, pure Go, no network). Positional args are files or
directories (walked recursively); `-`/no arg reads stdin.

```bash
vigolium kit secret-scan config.env .github/            # files + dirs
cat bundle.js | vigolium kit secret-scan -              # stdin
vigolium kit secret-scan -j --min-confidence high src/  # high-confidence JSON
```

| Flag | Effect |
|------|--------|
| `--min-confidence low\|medium\|high` | threshold (default `low`) |
| `--rule` / `--exclude-rule` | allow/deny by rule ID (comma-separated) |
| `--redact` | mask the value (prefix/suffix kept); **full value shown by default** |
| `--fail-on-match` | exit **3** when any secret is found (CI/agent gate) |
| `--max-file-size` | skip files larger than N bytes (default 16 MiB) |

JSON: `{files_scanned, count, matches:[{rule_id, rule_name, confidence, secret, entropy, file, line, start, end}]}`.
Reports *where* a secret is, never exploitability. Obvious placeholders
(`…EXAMPLE`, `123456…` sequences) are safelisted away.

## js-beautify

Unminify + unpack minified/bundled JavaScript with the embedded jstangle tool
(webcrack — no eval-based deobfuscation). Argument is a local file, an
`http(s)://` URL (fetched), or `-`/no arg for stdin.

```bash
vigolium kit js-beautify app.min.js                     # local file
vigolium kit js-beautify https://target.example/main.abc.js
cat bundle.js | vigolium kit js-beautify -              # stdin
vigolium kit js-beautify -j --extract app.min.js        # + extracted endpoints
vigolium kit js-beautify --modules ./out app.min.js     # one file per module
```

Default: beautified source to stdout (input emitted unchanged if it is neither
minified nor bundled). `--extract` also runs endpoint extraction.

| Flag | Effect |
|------|--------|
| `--extract` | also run endpoint extraction (a full analysis pass); included in `-j` output |
| `--modules <dir>` | write each recovered module to its own file under `<dir>`, at its recovered path |
| `-o, --output <file>` | write the beautified source to a file instead of stdout (`-` means stdout) |
| `--profile <p>` | `beautify` \| `endpoints` \| `discovery` \| `discovery-lite` \| `full` \| `inspect` \| `dom-security` \| `legacy`; default depends on `--extract` |
| `--unpack-modules` | unpack a bundle and re-scan each module for endpoints (implies `--extract`) |
| `--max-ast-nodes` | AST node budget before the analysis degrades to a per-module scan (0 = engine default, 500k) |
| `--max-input-mb` | **lowers** the input ceiling only; it cannot raise it above the service limit |
| `--deadline` | analysis deadline (0 = engine default, 60s) |
| `--timeout` | fetch timeout for a URL argument (default 30s) |

JSON: `{source, status, changed, format, module_count, module_paths?, bytes_in,
bytes_out, content, endpoints?, diagnostics?, output_path?, modules_dir?,
modules_written?}`. Endpoints carry `{url, method, params, body, headers,
cookies, extractor, confidence, module_path?}`.

**Read `status` before trusting `endpoints`.** `complete` is a clean AST pass.
`partial` means some stages did not run, and the `diagnostics` codes say which.
`failed` means the analysis produced nothing of its own.

**A big bundle returns `partial`, and that is the good outcome.** A script dense
enough to blow the AST budget cannot be parsed whole, so the engine unpacks it
and analyzes each module separately (`ast_budget_recovered_by_module_scan`).
Those endpoints carry `extractor: "bundle-module"` and a `module_path`, with real
methods, headers and body templates. A `partial` from that path is far better
data than a `complete` from a small file.

**An empty `method` means unresolved, not GET.** The `large-input-string-fallback`
extractor is a regex sweep over source that no analyzer could parse; it recovers
URLs only. Treat `confidence: "low"` plus an empty method as a lead to confirm,
never as a request to replay.

For a bundle of any size, prefer `--modules <dir>` over reading `content`: a
400-module document is one 5 MB blob, and the directory is greppable.

## oast (new / poll)

Generate interactsh out-of-band callback URLs and later poll for the
DNS/HTTP/SMTP interactions they received — the primitive behind blind
SSRF/XXE/RCE/log4shell confirmation. State lives in a **session file**, so
minting and polling are separate fire-and-forget invocations (no long-running
process).

```bash
vigolium kit oast new -n 2 --session run.yaml          # mint 2 URLs, save session
# ...inject the URL(s) somewhere, wait...
vigolium kit oast poll --session run.yaml --wait 30s -j   # drain interactions as JSON
vigolium kit oast poll --session run.yaml --deregister    # final drain + tear down
```

| Flag | Command | Effect |
|------|---------|--------|
| `-o/--session` | both | session file path (default `oast-session.yaml`) |
| `--server` / `--token` | `new` | interactsh server (default `oast.pro`) + auth for a self-hosted one |
| `-n/--count` | `new` | number of URLs to mint |
| `--wait` / `--interval` | `poll` | how long to poll / cadence (auto-shrunk below `--wait`) |
| `--deregister` | `poll` | after draining, destroy the session server-side and delete the file |

`poll` JSON: `{server, session, count, interactions:[{protocol, unique_id, full_id, remote_address, timestamp, raw_request, raw_response}]}`.
Point `--server`/`--token` at a self-hosted interactsh to keep callbacks
private. Keep the session file until you are done polling.

## harvest

Collect historically-known URLs for one or more domains from public archives and
indexes (Wayback, Common Crawl, AlienVault OTX, Arquivo by default; urlscan and
VirusTotal join when their key is configured under `external_harvester`). A URL
argument is reduced to its host; `-` reads domains one per line from stdin.

```bash
vigolium kit harvest target.example                     # one domain
vigolium kit harvest target.example shop.target.example -j
cat domains.txt | vigolium kit harvest -                # stdin
vigolium kit harvest --source wayback,commoncrawl target.example
```

Plain mode streams deduped URLs one per line; `-j` yields
`{domains, sources, count, urls}`. Same source set as the scan's
`external-harvest` phase.

## jwt-crack

Brute-force a JWT's HMAC signing secret (HS256/384/512) against a wordlist. A
token declaring an asymmetric alg (RS*/ES*/PS*) is tried under every HMAC
variant — the algorithm-confusion attack — and the match is labelled. Token is a
positional arg or `-`/stdin; a leading `Bearer ` is stripped.

```bash
vigolium kit jwt-crack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ...
echo "$TOKEN" | vigolium kit jwt-crack -
vigolium kit jwt-crack -w /path/rockyou.txt -p s3cr3t eyJ...
vigolium kit jwt-crack -j --fail-on-crack eyJ...
```

| Flag | Effect |
|------|--------|
| `-w/--wordlist` | built-in name (see `kit wordlist`) or a file path (default: embedded `jwt.secrets.list`, ~104k) |
| `-p/--secret` | inline candidate secret (repeatable) |
| `--fail-on-crack` | exit **3** when the secret is recovered |

JSON: `{alg, cracked, secret, matched_alg, candidates_tried, wordlist, header, payload}`.
The recovered secret is printed in full (that is the point).

## wordlist

List the embedded wordlists, or print one to stdout so it can be piped into
another tool (or `kit jwt-crack -w <name>`).

```bash
vigolium kit wordlist               # list available lists with entry/byte counts
vigolium kit wordlist -j            # the same, as JSON
vigolium kit wordlist fuzz          # print the fuzz list to stdout
vigolium kit wordlist jwt > jwt.txt # materialize (alias: jwt → jwt.secrets.list)
```

A name matches the embedded filename, its basename without extension, or the
`jwt` alias. Built-ins: `dir-short`, `dir-long`, `file-short`, `file-long`,
`fuzz`, `jwt.secrets.list`.

## payload

Print the built-in fuzzing payloads for one or more classes, one per line — the
same catalog behind `vigolium fuzz --class`.

```bash
vigolium kit payload --list              # list classes
vigolium kit payload --class sqli        # SQLi payloads
vigolium kit payload --class sqli,xss -j # two classes, as JSON
```

Classes: `cmdi`, `crlf`, `lfi`, `open_redirect`, `path_traversal`, `sqli`,
`ssrf`, `ssti`, `xss`, `xxe` (aliases like `sql`→`sqli`, `traversal`→
`path_traversal`). JSON: `{classes, count, payloads}`.

---

## Gotchas

- **Stateless by design.** Nothing lands in the project DB — pipe or redirect
  the output yourself. For findings that persist, use `scan`/`agent` instead.
- **`oast new` does not tear down the session.** Closing an interactsh client
  deregisters it server-side, so `new` mints + saves and exits without closing;
  the session stays alive for `poll`. Use `poll --deregister` to explicitly
  destroy it when done.
- **`oast --interval` is auto-shrunk** below `--wait` (the poll ticker fires
  only after one interval), so a short `--wait` still polls at least twice.
- **secret-scan safelists placeholders.** `AKIA…EXAMPLE` and `123456`-sequence
  tokens are treated as benign — use real-looking values to test.
- **`js-beautify` never exits non-zero for a degraded analysis.** A bundle that
  only half-analyzed still exits 0; `status` and `diagnostics` are the only
  signal. Check them, not the exit code.
- **`js-beautify --max-input-mb` cannot raise the ceiling.** The service takes
  the lower of this and its own limit (10 MiB by default), so a larger value is
  silently ignored. Raise `jstangle.hard_input_mb` in config instead.
- **Exit 3 gates.** `secret-scan --fail-on-match` and `jwt-crack --fail-on-crack`
  exit 3 on a hit. Otherwise 0 success, 1 error, 2 usage error — see
  [agent-loop.md → Exit codes](agent-loop.md#exit-codes).
