# Ingest Reference

> **Related:** [server.md](server.md) for the receiving server & recording proxy ·
> [scanning.md](scanning.md) for `-I`/`-i`/`-T` and the full input-format matrix ·
> [data.md](data.md) for browsing what landed · [agent-loop.md](agent-loop.md) for the fs mirror

Complete flag reference for `vigolium ingest` — parse traffic from a file, spec,
or stdin and store it as HTTP records, either into the local database or into a
running vigolium server.

`ingest` is the "load traffic, don't scan yet" primitive. To scan on receive,
add `-S` (local) or feed a server started with `-S` (see [server.md](server.md)).

## Table of Contents

- [ingest](#ingest)
- [Key global flags](#key-global-flags-for-ingest)
- [Local vs Remote Mode](#local-vs-remote-mode)
- [Input formats — one example each](#input-formats--one-example-each)
- [OpenAPI / Swagger spec flags](#openapi--swagger-spec-flags)
- [Combined examples](#combined-examples)

---

## ingest

**Usage:** `vigolium ingest [flags]`

Ingest HTTP requests into the database, either locally or via a remote server.
Responses are fetched by default (source `ingest-cli`); disable with
`--disable-fetch-response` to store request-only records.

### ingest-specific flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--server` | `-s` | string | — | Server URL for remote ingestion (omit for local mode) |

### Key Global Flags for Ingest

| Flag | Description |
|------|-------------|
| `-t <url>` | Base URL / target for the ingested data (required for specs that carry only paths) |
| `-i <file>` | Input file path (`-` for stdin) |
| `-I <format>` | Input format (`urls`, `openapi`, `swagger`, `wsdl`, `burp`, `curl`, `har`, `postman`, `nuclei`, `burpscope`) |
| `-T <file>` | Target-file: one target URL per line (for `urls`/`burpscope` only — a spec goes through `-i`) |
| `-S` | After ingesting, scan the records (local mode only) |
| `--spec-url` | Use server URLs from the OpenAPI/Swagger spec |
| `--spec-header` | HTTP header(s) for OpenAPI/WSDL requests (repeatable) |
| `--spec-var` | OpenAPI parameter / WSDL element values as `key=value` (repeatable) |
| `--spec-default` | Default value for required parameters (default: `1`) |
| `--disable-fetch-response` | Store request-only (don't fetch responses) |
| `--scope-origin` | Origin scope mode for filtering |
| `--no-tech-filter` | Disable the tech-stack allowlist (run every module regardless of detected stack; auto-on at `--intensity deep`) |
| `--no-waf-pacing` | Disable proactive CDN/WAF-edge pacing (reactive back-off after a WAF block still applies) |

> **`-i` vs `-T`:** `-i` reads a *spec / export* and expands it into requests;
> `-T` reads a *list of target URLs*, one per line. Pointing `-T` at a YAML spec
> makes every line of that file a target. The only formats that are genuinely
> target lists are `urls` and `burpscope`. See [scanning.md](scanning.md) for the
> full contract.

## Local vs Remote Mode

- **Local mode** (default): ingests directly into the local SQLite database and fetches HTTP responses.
- **Remote mode** (`--server <url>` / `-s`): sends records to a running vigolium server via `POST /api/ingest`. Set `VIGOLIUM_API_KEY` (or the server's `--alternative-ingest-key`) for auth.
- `-S` / `--scan-on-receive` is **ignored in remote mode** — the server decides whether to scan (start it with `-S`; see [server.md](server.md)).

## Input formats — one example each

`-I` selects the parser. OpenAPI/Swagger and WSDL auto-detect from file content,
so `-I` is optional for those.

```bash
# urls (default) — a single target, or a file of targets.
vigolium ingest -t https://target.example
vigolium ingest -T targets.txt

# OpenAPI 3.x — expand every operation into a request against the base URL.
vigolium ingest -I openapi -i openapi.yaml -t https://api.target.example

# Swagger 2.0.
vigolium ingest -I swagger -i swagger.json -t https://api.target.example

# WSDL 1.1 / SOAP — expand every bound operation into a SOAP POST. `-I` is
# optional (a .wsdl is content-sniffed); a .svc/.asmx URL auto-fetches its WSDL.
vigolium ingest -I wsdl -i service.wsdl -t https://api.target.example
vigolium ingest -i https://api.target.example/Service.svc

# Burp Suite XML export.
vigolium ingest -I burp -i burp-export.xml

# Burp project-config scope export → expands include rules into seed URLs.
vigolium ingest -I burpscope -i burp-scope.json

# cURL commands (one per line, or a pasted multi-line command).
vigolium ingest -I curl -i requests.txt

# HAR archive (browser DevTools / proxy export).
vigolium ingest -I har -i traffic.har

# Postman collection.
vigolium ingest -I postman -i collection.json

# Nuclei templates (extract request definitions).
vigolium ingest -I nuclei -i templates/

# stdin — pipe any supported format through `-i -`.
cat urls.txt         | vigolium ingest -i -
cat openapi.yaml     | vigolium ingest -I openapi -i - -t https://api.target.example
curl-command-here    | vigolium ingest -I curl  -i -
```

## OpenAPI / Swagger spec flags

Specs often need help resolving base URLs, auth, and required parameters:

```bash
# Use the server URLs declared inside the spec instead of -t.
vigolium ingest -I openapi -i spec.yaml --spec-url

# Attach auth headers for every generated request (repeatable).
vigolium ingest -I openapi -i spec.yaml -t https://api.target.example \
  --spec-header "Authorization: Bearer $TOKEN" \
  --spec-header "X-Api-Key: $KEY"

# Supply concrete values for path/query parameters, with a fallback default.
vigolium ingest -I openapi -i spec.yaml -t https://api.target.example \
  --spec-var userId=42 --spec-var status=active --spec-default 1
```

### WSDL / SOAP

A WSDL becomes one SOAP POST per bound operation (SOAP 1.1 with a `SOAPAction`
header, or SOAP 1.2 with `action=` in the content type; document/literal and rpc
styles). The XML body is synthesized from the WSDL's inline XSD types and is
fuzzable by the normal active modules. `--spec-header` and `--spec-var` apply
here too — `--spec-var` overrides a leaf element's placeholder value by its local
name.

```bash
# Endpoint comes from the WSDL's <soap:address>; -t overrides only the host
# (scheme+host from -t, service path from the WSDL) to keep traffic in scope.
vigolium ingest -I wsdl -i service.wsdl -t https://soap.target.example \
  --spec-header "Authorization: Bearer $TOKEN" \
  --spec-var userName=admin

# A live WCF (.svc) or ASMX (.asmx) endpoint — the WSDL is fetched automatically
# (?singleWsdl / ?WSDL), so no local file is needed.
vigolium ingest -i https://soap.target.example/Service.svc
```

WSDL 2.0 and unresolved multi-file `<import>` are not supported — point at a
`.svc` `?singleWsdl` (which inlines schemas) when a WSDL splits its types across
files.

## Combined examples

```bash
# Ingest a spec locally, then auto-scan the generated requests.
vigolium ingest -t https://api.example.com -I openapi -i spec.yaml -S

# Ingest a Burp export request-only (no response fetching — fast, offline).
vigolium ingest -I burp -i export.xml --disable-fetch-response

# Push a spec to a running server for it to scan (remote mode).
VIGOLIUM_API_KEY=… vigolium ingest -s http://localhost:9002 -I openapi -i spec.yaml

# Ingest into a specific project.
vigolium ingest -I har -i traffic.har --project-name acme-audit

# Load traffic, then browse it (see data.md).
vigolium ingest -I burp -i export.xml
vigolium traffic --source ingest-cli --tree
```

After ingesting, records are queryable with `vigolium traffic` (filter by
`--source ingest-cli` / `ingest-server` / `ingest-proxy`) — see
[data.md](data.md).
