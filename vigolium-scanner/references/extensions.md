# Writing JavaScript Extensions

> **Related:** [data.md](data.md) for the `extensions` / `js` commands

Guide for writing custom vigolium scanner modules as JavaScript extensions.

## Table of Contents

- [Overview](#overview)
- [Extension Structure](#extension-structure)
- [Running Extensions](#running-extensions)
- [Module Types](#module-types)
- [Context Object](#context-object)
- [API Reference Summary](#api-reference-summary)
- [Creating Findings](#creating-findings)
- [Examples](#examples)
- [YAML Extensions](#yaml-extensions)

---

## Overview

Vigolium extensions are JavaScript files that implement scanner modules using the embedded Sobek (Grafana k6) engine. Extensions can be:

- **Active modules**: Send modified requests to detect vulnerabilities
- **Passive modules**: Analyze existing request/response pairs without sending traffic
- **Pre/post hooks**: Filter or rewrite each input item before modules see it (`pre_hook`), or each result before it is stored (`post_hook`); see [Pre-Hook and Post-Hook](#pre-hook-and-post-hook)

Extensions have access to the full `vigolium.*` API for HTTP requests, database queries, parsing, AI-augmented analysis, OAST (out-of-band) testing, and finding creation.

---

## Extension Structure

Every extension exports a `module.exports` object with metadata and scan functions:

```javascript
module.exports = {
  // Required metadata
  id: "my-custom-check",           // Unique module ID
  name: "My Custom Check",         // Human-readable name
  type: "passive",                 // "active" or "passive"
  severity: "medium",              // "info", "low", "medium", "high", "critical", "suspect"
  confidence: "certain",           // "certain", "firm", "tentative"

  // Optional metadata
  description: "Checks for ...",
  confirmationCriteria: "...",     // how a finding is confirmed; string or array of lines
  scope: "response",               // "request", "response", "both" (passive only)
  tags: ["custom", "light"],       // Module tags for filtering
  scanTypes: ["per_request"],      // "per_request", "per_host", "per_insertion_point"

  // Scan functions (implement one or more)
  scanPerRequest: function(ctx) {
    // Analyze the request/response pair
    // Return a finding object, an array of them, or null
  },

  scanPerHost: function(ctx) {
    // Analyze all records for a host
  },

  scanPerInsertionPoint: function(ctx, insertion) {
    // Test one insertion point (active modules).
    // insertion = {name, baseValue, type, buildRequest(payload) -> raw request string}
  }
};
```

A returned finding may override the module defaults per result: `confidence`,
`tags`, `cwe` (e.g. `"CWE-79"`), `fuzzingParameter` (or `param`), and
`evidence` (an array of strings or `{request, response, note}` objects).

For editor autocompletion, `vigolium ext types -o <dir>` writes `vigolium.d.ts`
(also written by `vigolium ext preset`); reference it with
`/// <reference path="./vigolium.d.ts" />`.

---

## Running Extensions

```bash
# Run a single extension against a target
vigolium run extension -t https://example.com --ext custom-check.js

# Short alias
vigolium run ext -t https://example.com --ext custom-check.js

# Run during a full scan (alongside built-in modules)
vigolium scan -t https://example.com --ext custom-check.js

# Run only extensions, skip built-in modules
vigolium scan -t https://example.com --only extension --ext custom-check.js

# Run extensions together with the built-in dynamic-assessment modules
vigolium scan -t https://example.com --only dynamic-assessment,extension --ext custom-check.js

# Load multiple extensions
vigolium scan -t https://example.com --ext check1.js --ext check2.js

# Load all extensions from a directory
vigolium scan -t https://example.com --ext-dir ./my-extensions/

# Quick-test JS code inline
vigolium ext eval 'vigolium.log.info("hello")'
vigolium ext eval --ext-file script.js

# Install preset examples
vigolium ext preset

# View full API docs
vigolium ext docs
vigolium ext docs --example
vigolium ext docs http   # filter by namespace
vigolium ext docs -j     # API catalog as JSON

# TypeScript declarations for the API (stdout, or -o <file|dir>)
vigolium ext types -o ./my-extensions/
```

### Lint, and what happens when an extension does not load

```bash
vigolium ext lint custom-check.js        # exit 1, error.code lint_failed, on errors
vigolium ext lint ./my-extensions/ -j    # one document: items = files, summary {files, errors, warnings}
vigolium ext lint custom-check.js --strict   # warnings fail too
cat check.vgm.yaml | vigolium ext lint --stdin   # sniffed; --stdin-kind js|ts|yaml to force
```

- Lint runs the top-level code in the same side-effect-free VM the scan loader uses. Pure APIs (`parse.*`, encoding/hash/regex utils, `log.*`, `payloads()`, `config.*`) run; effectful ones (`http.*`, `utils.exec`/`writeFile`, `db.*`, `ingest.*`, `agent.*`, `oast.*`, `scan.createFinding`) are stubbed and reported as `top-level-effect` warnings with a line. Every namespace exists at load time, so `if (vigolium.db)` feature detection belongs in a handler.
- Errors: `syntax`, `transpile`, `metadata` (top-level code threw / no `type`), `no-exports` (unless the file has a `// vigolium:script` line), `missing-handler` (a declared `scanTypes` entry, default `per_request`, or a hook's `execute` is not a function), `invalid-type`, `bad-confidence`, `yaml-parse`, `yaml-schema`, `yaml-unknown-key`.
- Each `-j` issue is `{severity, line, col, message, rule}`; `ok` is false exactly when there are errors.
- A file passed with `--ext` (or `custom_dir`) that fails to load stops the scan before any request: exit `2`, `error.code` `extension_load_failed`, naming the file and stage. `--ext-allow-failures` scans without it. A failure in the extension directory is a warning only.
- Every load failure emits `extension.failed` (`module_id`, `path`, `stage`, `explicit`, `message`) on `--events` and appears in `scan.finished.extensions_failed`.
- YAML unknown keys only warn at load time unless `extensions.yaml_strict: true` (the default flips to true next minor).

### Write an extension, then `ext test` it

The loop: write the file, `ext lint` it, run it against one stored record with `ext run`, then pin what it should and should not report with fixtures and `ext test`. None of these scan anything.

```bash
# Run it against a stored record (offline: its requests are refused and counted)
vigolium traffic --compact --fields uuid,url -j
vigolium ext run --ext check.js --record-uuid 3f2a9c1e -j
vigolium ext eval --ext check.js --record-uuid 3f2a9c1e -j     # same command, same document
vigolium ext run --ext probe.js --record-uuid 3f2a9c1e --handler scanPerInsertionPoint --insertion URL_PARAM:id
vigolium ext run --ext probe.js --record-uuid 3f2a9c1e --allow-network   # really send its requests

# Fixtures next to the extension, then run them
vigolium ext test ./my-extensions/              # TAP-like; exit 1 if any case fails
vigolium ext test ./my-extensions/check.js -j   # items = cases; passed / failed / skipped
```

- `ext run -j`: `items` are the results (`module_id, name, severity, confidence, url, matched, description, param, payload, request, response`), with the module's defaults filled in as a scan fills them. Top level: `module_id, record_uuid, handler, handlers, calls, errors[{handler, insertion, message}], network_allowed, network_calls_blocked, duration_ms`. A handler that threw makes `ok` false and exits `1`; each error message ends with the script position (`check.js:12:9`). An unknown record exits `2` with `record_not_found`, and an extension that does not load exits `2` with `extension_load_failed`.
- Offline is the default. Every `vigolium.http.*` call throws `network disabled in ext eval; pass --allow-network`, and every refused request counts in `network_calls_blocked`. The database is opened read-only: `vigolium.db.*` reads work, while `ctx.record.annotate`/`addRemarks` and `vigolium.ingest.*` return false. `vigolium.oast` and `vigolium.agent` are not available.
- Without `--handler`, every handler the `scanTypes` declare runs. `scanPerInsertionPoint` runs on every point the module accepts, unless `--insertion TYPE:name` picks one. An unknown point exits `2` and lists the available ones.

Fixture layout:

```text
my-extensions/
  check.js
  check.fixtures/
    weak.http                # a raw request, a blank line, a raw response
    weak.expect.json         # {"findings": [{"name~": "weak csp", "severity": "medium"}]}
    clean.http               # or put the expectation in the file: // expect: {"findings": 0}
  probe.js
  probe.fixtures/
    reflected.http           # // insertion: URL_PARAM:q
    reflected.responses/     # canned answers to the extension's own requests
      search.http            # // match: VIGOLIUM, then "GET /search", a blank line, the response
```

```http
// expect: {"findings": [{"name~": "weak content-security-policy", "severity": "medium", "confidence": "firm"}]}
GET / HTTP/1.1
Host: app.example.com

HTTP/1.1 200 OK
Content-Type: text/html
Content-Security-Policy: default-src *; script-src 'unsafe-inline'

<html></html>
```

- **Expectation.** `findings` is either a count or a list of matchers. A list is matched in any order and the counts must agree. Matcher keys are `name~` (substring), `name` (exact), `severity` and `confidence`, all case-insensitive. `errors` defaults to `0`. `network_calls_blocked`, `mock_requests`, `handler` and `insertion` are optional. An unknown key is an error, so a typo cannot silently expect nothing.
- **Directives** (leading `//` lines of a case): `expect`, `handler`, `insertion`, and `url` (the request's origin; default `http://<Host>`). LF line endings are fine, and a declared `Content-Length` is corrected to the body.
- **Mock responses.** A case with `<case>.responses/` runs against an in-process mock on 127.0.0.1, and the case's `Host` points at it. Each response file holds `METHOD /path` (add `?query` to match the full target), an optional `// match: <text>` that the target or body must contain, a blank line, and then the response. Unmatched requests get a 404 and are listed in `unmatched_requests`. A case without that directory runs offline. `ext test` opens no database.
- Extensions with no fixtures, and hooks, are reported as `skipped`. A `<name>.fixtures` directory with no extension fails. The bundled presets ship fixtures (`public/presets/extensions/*.fixtures`), and `make test-unit` runs them.

### Module selection applies to extensions

Extension modules have IDs `ext-<id>` and follow the same selection flags as built-in modules:

| Flag | Effect on extensions |
| --- | --- |
| `--passive-only` | Active extensions do not run. Pre/post hooks still do. |
| `--no-passive` | Passive extensions do not run. |
| `--module-id ext-a` | Only the named extensions run. An `ext-` ID that matches no loaded extension exits `2` (`usage_error`) before any request. |
| `-m csp`, `-m ext-csp` | Fuzzy match on extension ID and name, like built-ins. |
| `--module-tag headers` | Selects extensions whose `tags` include it. |
| `--module-id`/`-m`/`--module-tag` naming only built-ins | Extensions from `--ext` (or `custom_dir`) still run; extensions auto-loaded from the extension directory do not. |

`-m` and `--module-tag` narrow only active modules on `scan`, so passive extensions keep running there. The dynamic-assessment log line reports the selection, for example `extensions: 1 active, 2 passive selected; 1 skipped (1 directory-loaded, narrowed selection)`.

```bash
# Run one extension and nothing else
vigolium scan -t https://example.com --ext-dir ./my-extensions/ --module-id ext-my-check
```

---

## Module Types

### Passive Module

Analyzes existing traffic without sending new requests. Good for pattern detection, data exposure, misconfiguration checks.

```javascript
module.exports = {
  id: "header-check",
  name: "Missing Security Headers",
  type: "passive",
  severity: "low",
  confidence: "certain",
  scope: "response",
  tags: ["headers", "misconfiguration", "light"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    if (!ctx.response) return null;
    var headers = ctx.response.headers;

    var missing = [];
    if (!headers.get("x-frame-options") && !headers.get("content-security-policy")) {
      missing.push("X-Frame-Options or CSP frame-ancestors");
    }
    if (!headers.get("strict-transport-security")) {
      missing.push("Strict-Transport-Security");
    }

    if (missing.length === 0) return null;

    return {
      url: ctx.request.url,
      name: "Missing Security Headers: " + missing.join(", "),
      severity: "low",
      description: "The response is missing recommended security headers."
    };
  }
};
```

Header objects (`ctx.request.headers`, `ctx.response.headers`, every `vigolium.http.*` response) are case-insensitive: `headers.get("x-frame-options")` is the idiom, and `headers["X-Frame-Options"]` reads the same value. `Object.keys(headers)` returns lowercase names. A repeated header reads as its values joined with `", "`, except `set-cookie`, which reads as the last value; use `headers.getAll("set-cookie")` for all of them. `headers.entries()` returns original-case `[name, value]` pairs in wire order, and `headers.raw()` returns the pre-0.7 original-case object.

### Active Module

Sends modified requests to test for vulnerabilities. Use `vigolium.http` to send requests.

```javascript
module.exports = {
  id: "custom-path-traversal",
  name: "Path Traversal Check",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["lfi", "traversal"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    var payloads = ["../../../etc/passwd", "....//....//....//etc/passwd"];
    var parsed = vigolium.parse.url(ctx.request.url);
    if (!parsed) return null;

    for (var i = 0; i < payloads.length; i++) {
      var testUrl = parsed.scheme + "://" + parsed.host + "/" + payloads[i];
      var resp = vigolium.http.get(testUrl, {
        headers: ctx.request.headers
      });

      if (resp.status === 200 && resp.body.indexOf("root:") !== -1) {
        return {
          url: testUrl,
          name: "Path Traversal",
          severity: "high",
          matched: "root:",
          request: "GET " + testUrl,
          response: resp.raw
        };
      }
    }
    return null;
  }
};
```

### Pre-Hook and Post-Hook

Hooks are not modules: they do not scan, they transform what flows through the
pipeline. Both export `type: "pre_hook"` or `type: "post_hook"` plus a single
`execute` function (no `scanPerRequest`/`scanPerHost`), and load the same way as
modules (`--ext`, `--ext-dir`, or the configured extensions dir).
`vigolium ext example pre_hook` / `ext example post_hook` prints a starter.
Runtime contract: `pkg/jsext/hooks.go`.

**`pre_hook`** runs once on every input item **before any module sees it**.
`execute(request)` receives `{raw, method, url, headers}` (`url` is the full
target URL; `headers` is a name → value object). What it returns decides the
item's fate:

| Return | Effect |
|---|---|
| `null` / `undefined` | **Skip the item**: no module (active or passive) sees it |
| `{ raw: "GET / HTTP/1.1\r\n..." }` | Replace the whole raw request (original host/service kept). `raw` wins over `headers` |
| `{ headers: {...} }` | Add or replace these headers on the original request |
| anything else (e.g. `request` unchanged) | Pass through as-is |

```javascript
module.exports = {
  id: "skip-static-assets",
  type: "pre_hook",
  execute: function(request) {
    var path = (vigolium.parse.url(request.url) || {}).path || request.url;
    if (/\.(css|png|jpe?g|gif|svg|ico|woff2?|ttf|map)$/i.test(path)) return null;
    return { headers: { "X-Scan-Source": "vigolium" } };
  }
};
```

**`post_hook`** runs on every result a module emits, **before** dedup and
storage. `execute(result)` receives `{moduleId, url, host, matched, request,
response, info: {name, description, severity, confidence}}` (`templateId` is an
alias of `moduleId`). Return `null`/`undefined` to **drop** the result, or an
object whose `url`, `matched`, `info.name`, `info.description` and
`info.severity` are copied back onto the result. Any other field you change
(`info.confidence`, `request`, `host`, …) is ignored.

```javascript
module.exports = {
  id: "escalate-admin-paths",
  type: "post_hook",
  execute: function(result) {
    if (result.url.indexOf("/admin") === -1) return result;
    result.info.severity = "high";
    result.info.name = result.info.name + " [admin]";
    return result;
  }
};
```

Hooks chain in load order: each one gets the previous one's output, and the
first `null` stops the chain. **A hook that throws (or whose `execute` is not a
function) is skipped with a warning and the item continues unchanged**, so a
broken hook never filters anything. Watch the log for `Pre-hook error` /
`Post-hook error` rather than assuming the hook ran.

---

## Context Object

The `ctx` object passed to scan functions contains:

```typescript
{
  request: {
    raw: string,        // Full raw HTTP request
    method: string,     // HTTP method
    url: string,        // Full URL
    headers: Record<string, string>
  },
  response: {
    status: number,     // HTTP status code
    body: string,       // Response body
    raw: string,        // Full raw HTTP response
    headers: Record<string, string>
  },
  record: {
    uuid: string,       // Database UUID of the HTTP record
    annotate(patch),    // Update risk_score/remarks
    addRiskScore(delta),// Increment risk score
    addRemarks(remarks) // Append remarks
  }
}
```

The global `vigolium.record` also provides access to the current record context.

---

## API Reference Summary

### vigolium.log
- `info(msg)`, `warn(msg)`, `error(msg)`, `debug(msg)`

### vigolium.utils
- Encoding: `base64Encode/Decode`, `urlEncode/Decode`, `htmlEncode/Decode`
- Hashing: `sha1`, `sha256`, `md5`
- Random: `randomString(len)`
- Regex: `regexMatch(str, pattern)`, `regexExtract(str, pattern)`, `regexFindAll(str, pattern)` — all non-overlapping matches, or null
- File I/O: `readFile`, `readLines`, `writeFile`, `mkdir`, `glob`
- URL: `parse_url(url, format)`, `pathToTemplate(path)`, `hasDynamicSegment(path)`
- Params: `toSet(csv)`, `extractParamNames(str)`
- Diff/similarity: `diff(a, b)`, `similarity(a, b)`, `diffResponses(respA, respB)`
- HTML: `cssSelect(html, selector)` — CSS selector queries on HTML
- Token extraction: `extractToken(response, rules)` — extract tokens from HTTP responses
- JWT: `jwtDecode(token)`, `jwtEncode(payload, opts?)`, `jwtExpired(token)`
- TOTP: `totpCode(secret)` — generate a TOTP code from a base32 secret (RFC 6238)
- Multipart: `multipart(fields)` — build multipart/form-data bodies
- Anomaly: `detectAnomaly(responses)`
- Other: `sleep(ms)`, `exec(cmd)`, `getEnv`, `setEnv`, `jsonExtract`

### vigolium.parse
- `url(str)`, `request(raw)`, `response(raw)`
- `headers(str)`, `cookies(str)`, `query(str)`, `json(str)`, `form(body)`
- `html(str)` — parse HTML into forms, links, scripts, meta tags

### vigolium.http
- Basic: `get(url, opts?)`, `post(url, body, opts?)`, `request(opts)`, `send(rawRequest)`
- Build: `buildRequest(rawRequest, overrides)` — clone and modify a raw request
- Session: `session(opts?)` — persistent HTTP session with shared cookie jar
- Login: `login(opts)` — send credentials, extract tokens, return authenticated session
- Batch: `batch(requests, opts?)` — parallel request execution
- Replay: `replay(rawRequest, variations)` — replay with multiple variations
- Sequence: `sequence(steps)` — multi-step workflows with variable extraction
- Auth testing: `authTest(opts)` — IDOR/BOLA testing across privilege levels
- Session pool: `sessionPool(configs)` — named session management
- CSRF: `csrf(url, opts?)` — extract CSRF tokens from pages
- OAuth: `followAuth(opts)` — OAuth2 authentication flows
- Retry: `retry(request, opts?)` — retry with configurable backoff
- GraphQL: `graphql(url, opts)`, `graphqlSchema(url, opts?)` — queries and introspection
- Cache: `cache(opts?)`, `clearCache()`, `cachedGet(url, opts?)`, `cachedRequest(opts)`

### vigolium.scan
- `listModules()`, `listModuleTags()`, `listModulesByTag(tag)`
- `isInScope(host, path)`, `getScope()`, `setScope(scope)`
- `createFinding(finding)`, `getCurrentScan()`, `startNewScan(opts)`
- `scanRecords(opts)` — queue a scan for existing records by UUIDs

### vigolium.ingest
- `url(url)`, `urls(content)`, `curl(command)`, `raw(request, response?)`
- `openapi(spec, opts?)`, `postman(collection)`

### vigolium.agent (AI-augmented)
- `ask(prompt, opts?)` — single prompt → text response
- `chat(messages, opts?)` — conversation → text response
- `complete(opts)` — full control with JSON schema support
- `generatePayloads(opts)` — generate security test payloads
- `analyzeResponse(opts)` — analyze HTTP exchange for vulnerabilities
- `confirmFinding(opts)` — verify if a finding is a true positive
- `run(opts)` — run a full agent backend subprocess

### vigolium.oast (Out-of-Band Testing)
- `enabled()` — check if OAST service is active
- `payload(targetURL?, paramName?, injectionType?)` — generate a unique callback URL
- `url({class?, target?, param?, label?})` — callback URL string (or null) rated as `class` (`ssrf`, `xxe`, `cmdi`, `sqli`, `ssti`, `jndi`, `esi`, … see `OASTClass` in vigolium.d.ts; default `generic`); an unknown class throws and fails `ext lint`
- `poll(timeoutMs?)` — wait and return all OAST interactions

### vigolium.db
- `records.query(filters?)`, `records.get(uuid)`, `records.getRelated(uuid)`
- `records.annotate(uuid, patch)`, `records.grouped(opts?)` — group by path template
- `findings.query(filters?)`, `findings.get(id)`, `findings.getByRecord(uuid)`
- `findings.create(finding)`
- `compareResponses(records)` — anomaly detection across records

### vigolium.payloads(type)
- Returns built-in payload wordlists by vulnerability type
- Types: `"xss"`, `"sqli"`, `"ssti"`, `"ssrf"`, `"lfi"`, `"path_traversal"`, `"xxe"`, `"cmdi"`, `"open_redirect"`, `"crlf"`

### vigolium.mcp (Model Context Protocol)
- `client(url, opts?)` — MCP client over a base URL (auto-tracks `Mcp-Session-Id`); the handle exposes `initialize()`, `notifyInitialized()`, `listTools()`, `callTool(name, args)`, `readResource(uri)`, plus raw `request`/`notify`/`send` escape hatches
- `parseSse(body)` — parse an SSE response body into discrete events
- `detect(req, resp)` — true when the request/response pair carries strong MCP indicators
- `buildRequest(method, params?, opts?)` — marshal a JSON-RPC 2.0 envelope (feed into `vigolium.http.send()`)

### vigolium.config
- Read-only config values: `vigolium.config["key"]`

---

## Creating Findings

Return a finding object from scan functions:

```javascript
return {
  url: "https://example.com/vuln",     // Required: URL where the issue was found
  name: "Finding Title",               // Required: short title
  severity: "high",                     // Optional: overrides module default
  description: "Detailed explanation",  // Optional
  matched: "pattern found in response", // Optional: matched evidence
  request: "raw request string",        // Optional: HTTP request
  response: "raw response string",      // Optional: HTTP response
  additional_evidence: ["extra1"]       // Optional: array of extra evidence
};
```

Or use `vigolium.scan.createFinding()` for more control:

```javascript
vigolium.scan.createFinding({
  url: "https://example.com/vuln",
  name: "Custom Finding",
  severity: "high",
  description: "...",
  request: rawReq,
  response: rawResp
});
```

---

## Examples

### AI-Augmented Active Scanner

```javascript
module.exports = {
  id: "ai-xss-check",
  name: "AI-Augmented XSS Scanner",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["xss", "ai"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    // Generate context-aware payloads using AI
    var payloads = vigolium.agent.generatePayloads({
      type: "xss",
      context: "HTML attribute",
      technology: "React",
      count: 5
    });

    var parsed = vigolium.parse.url(ctx.request.url);
    if (!parsed || !parsed.query) return null;

    for (var i = 0; i < payloads.length; i++) {
      var testUrl = parsed.scheme + "://" + parsed.host + parsed.path + "?q=" + encodeURIComponent(payloads[i]);
      var resp = vigolium.http.get(testUrl);

      // Use AI to analyze the response
      var analysis = vigolium.agent.analyzeResponse({
        request: "GET " + testUrl,
        response: resp.raw,
        vulnerability_type: "xss",
        payload: payloads[i]
      });

      if (analysis.vulnerable && analysis.confidence !== "low") {
        return {
          url: testUrl,
          name: "XSS via AI analysis",
          severity: "high",
          matched: analysis.evidence,
          description: analysis.details
        };
      }
    }
    return null;
  }
};
```

### Database-Driven IDOR Detector

```javascript
module.exports = {
  id: "idor-detector",
  name: "IDOR Detection via Response Comparison",
  type: "passive",
  severity: "suspect",
  confidence: "tentative",
  tags: ["idor", "bola", "access-control"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    if (!ctx.record.uuid) return null;
    var parsed = vigolium.parse.url(ctx.request.url);
    if (!parsed || !vigolium.utils.hasDynamicSegment(parsed.path)) return null;

    // Get related records (same path template, different IDs)
    var related = vigolium.db.records.getRelated(ctx.record.uuid, { limit: 5 });
    if (related.length < 2) return null;

    // Compare responses for anomalies
    var result = vigolium.db.compareResponses(related);
    if (result.all_similar || result.variant_count === 0) return null;

    ctx.record.addRiskScore(30);
    ctx.record.addRemarks(["idor-candidate: " + result.summary]);

    return {
      url: ctx.request.url,
      name: "Potential IDOR: " + result.summary,
      severity: "suspect",
      description: "Response divergence detected across records with same path template."
    };
  }
};
```

### Session-Based IDOR Testing with authTest

```javascript
module.exports = {
  id: "session-idor-test",
  name: "Session-Based IDOR Test",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["idor", "bola", "access-control"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    var parsed = vigolium.parse.url(ctx.request.url);
    if (!parsed || !vigolium.utils.hasDynamicSegment(parsed.path)) return null;

    // Create sessions for different privilege levels
    var pool = vigolium.http.sessionPool({
      admin: {
        url: parsed.scheme + "://" + parsed.host + "/api/login",
        body: JSON.stringify({ username: "admin", password: vigolium.config.admin_pass }),
        headers: { "Content-Type": "application/json" },
        extract: [{ source: "json", path: "token", apply_as: "Authorization: Bearer {value}" }]
      },
      user: {
        url: parsed.scheme + "://" + parsed.host + "/api/login",
        body: JSON.stringify({ username: "user", password: vigolium.config.user_pass }),
        headers: { "Content-Type": "application/json" },
        extract: [{ source: "json", path: "token", apply_as: "Authorization: Bearer {value}" }]
      },
      unauthenticated: {}
    });

    // Test IDOR across sessions
    var results = vigolium.http.authTest({
      sessions: [pool.get("admin"), pool.get("user"), pool.get("unauthenticated")],
      records: [ctx.record.uuid],
      method: "replay"
    });

    for (var i = 0; i < results.length; i++) {
      if (results[i].vulnerability !== "none") {
        return {
          url: ctx.request.url,
          name: "IDOR: " + results[i].vulnerability.toUpperCase(),
          severity: "high",
          matched: "Unauthorized access detected across privilege levels",
          description: "Confidence: " + results[i].confidence
        };
      }
    }
    return null;
  }
};
```

### Out-of-Band (OAST) Detection

```javascript
module.exports = {
  id: "oast-ssrf-check",
  name: "OAST SSRF Detection",
  type: "active",
  severity: "high",
  confidence: "certain",
  tags: ["ssrf", "oast"],
  scanTypes: ["per_insertion_point"],

  scanPerInsertionPoint: function(ctx, insertion) {
    if (!vigolium.oast.enabled()) return null;

    // Generate a unique OAST callback URL; class "ssrf" decides how a callback is rated
    var cb = vigolium.oast.url({class: "ssrf", target: ctx.request.url, param: insertion.name});
    if (!cb) return null;

    // Inject the callback URL as the parameter value
    var req = insertion.buildRequest(cb);
    vigolium.http.send(req);

    // Wait for out-of-band interactions
    var interactions = vigolium.oast.poll(5000);
    for (var i = 0; i < interactions.length; i++) {
      if (interactions[i].parameter_name === insertion.name) {
        return [{
          url: ctx.request.url,
          name: "SSRF via " + insertion.name,
          severity: "high",
          matched: "OAST callback received from " + interactions[i].remote_address,
          description: "Server-side request forgery confirmed via out-of-band interaction."
        }];
      }
    }
    return null;
  }
};
```

### Multi-Step Auth Flow with Sequence

```javascript
module.exports = {
  id: "auth-flow-test",
  name: "Multi-Step Authentication Flow Test",
  type: "active",
  severity: "medium",
  confidence: "firm",
  tags: ["auth", "session"],
  scanTypes: ["per_host"],

  scanPerHost: function(ctx) {
    var host = vigolium.parse.url(ctx.request.url).host;

    // Execute a multi-step login flow with variable extraction
    var result = vigolium.http.sequence([
      {
        method: "GET",
        url: "https://" + host + "/login",
        extract: {
          csrf: { source: "regex", pattern: 'name="csrf_token" value="([^"]+)"' }
        }
      },
      {
        method: "POST",
        url: "https://" + host + "/login",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: "username=test&password=test&csrf_token={{csrf}}",
        extract: {
          session: { source: "cookie", name: "session_id" }
        }
      },
      {
        method: "GET",
        url: "https://" + host + "/api/profile",
        headers: { "Cookie": "session_id={{session}}" },
        condition: "{{session}} != ''"
      }
    ]);

    if (result.success && result.responses[2] && result.responses[2].status === 200) {
      // Check if profile data leaks sensitive info
      var body = result.responses[2].body;
      if (/password|secret|api_key|private_key/.test(body)) {
        return {
          url: "https://" + host + "/api/profile",
          name: "Sensitive Data in Profile Endpoint",
          severity: "medium",
          matched: "Profile response contains sensitive field names"
        };
      }
    }
    return null;
  }
};
```

### GraphQL Introspection Check

```javascript
module.exports = {
  id: "graphql-introspection",
  name: "GraphQL Introspection Enabled",
  type: "active",
  severity: "low",
  confidence: "certain",
  tags: ["graphql", "information-disclosure"],
  scanTypes: ["per_host"],

  scanPerHost: function(ctx) {
    var parsed = vigolium.parse.url(ctx.request.url);
    var endpoints = ["/graphql", "/api/graphql", "/v1/graphql"];

    for (var i = 0; i < endpoints.length; i++) {
      var url = parsed.scheme + "://" + parsed.host + endpoints[i];
      var schema = vigolium.http.graphqlSchema(url);
      if (schema) {
        return {
          url: url,
          name: "GraphQL Introspection Enabled",
          severity: "low",
          matched: "Full schema available via introspection",
          description: "GraphQL introspection is enabled, exposing the full API schema."
        };
      }
    }
    return null;
  }
};
```

### JWT Manipulation Check

```javascript
module.exports = {
  id: "jwt-none-alg",
  name: "JWT None Algorithm Bypass",
  type: "active",
  severity: "critical",
  confidence: "firm",
  tags: ["jwt", "auth-bypass"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    // Look for JWT in Authorization header
    var auth = ctx.request.headers.get("authorization");
    if (!auth || auth.indexOf("Bearer ") !== 0) return null;

    var token = auth.substring(7);
    var decoded = vigolium.utils.jwtDecode(token);
    if (!decoded) return null;

    // Forge a token with "none" algorithm
    var forged = vigolium.utils.jwtEncode(decoded.payload, { algorithm: "none" });

    // Replay the request with the forged token
    var resp = vigolium.http.request({
      method: ctx.request.method,
      url: ctx.request.url,
      headers: Object.assign({}, ctx.request.headers, {
        "Authorization": "Bearer " + forged
      })
    });

    // If the response is similar, the server accepts unsigned tokens
    if (resp.status === ctx.response.status) {
      var sim = vigolium.utils.similarity(ctx.response.body, resp.body);
      if (sim > 0.9) {
        return {
          url: ctx.request.url,
          name: "JWT None Algorithm Accepted",
          severity: "critical",
          matched: "Server accepts JWT with 'none' algorithm",
          request: "Authorization: Bearer " + forged.substring(0, 50) + "...",
          response: resp.raw
        };
      }
    }
    return null;
  }
};
```

### Built-in Payloads with Caching

```javascript
module.exports = {
  id: "cached-sqli-check",
  name: "SQL Injection (with response caching)",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["sqli"],
  scanTypes: ["per_insertion_point"],

  scanPerInsertionPoint: function(ctx, insertion) {
    // Enable caching to avoid redundant baseline requests
    vigolium.http.cache({ ttl_ms: 30000 });

    // Use built-in payloads
    var payloads = vigolium.payloads("sqli");

    // Get cached baseline response
    var baseline = vigolium.http.cachedGet(ctx.request.url);

    for (var i = 0; i < payloads.length; i++) {
      var req = insertion.buildRequest(payloads[i]);
      var resp = vigolium.http.send(req);
      if (!resp) continue;

      // Check for SQL error patterns
      if (/SQL|syntax|mysql|pg_|ORA-|SQLSTATE/.test(resp.body)) {
        return [{
          url: ctx.request.url,
          name: "SQL Injection in " + insertion.name,
          severity: "high",
          matched: resp.body.match(/SQL[^\n]{0,100}|syntax[^\n]{0,100}/)[0],
          request: req,
          response: resp.raw
        }];
      }

      // Check for response divergence indicating boolean-based SQLi
      if (baseline) {
        var sim = vigolium.utils.similarity(baseline.body, resp.body);
        if (sim < 0.5 && resp.status !== baseline.status) {
          var confirmed = vigolium.agent.confirmFinding({
            name: "Boolean-based SQL Injection",
            request: req,
            response: resp.raw,
            matched: "Response divergence: similarity=" + sim.toFixed(2),
            baseline_response: baseline.raw
          });
          if (confirmed.confirmed) {
            return [{
              url: ctx.request.url,
              name: "SQL Injection (boolean-based) in " + insertion.name,
              severity: "high",
              matched: confirmed.reasoning
            }];
          }
        }
      }
    }
    return null;
  }
};
```

### Multi-Version Extensions (Multiple Detection Techniques)

For the same vulnerability sink, create multiple extension files using different detection techniques. This maximizes coverage — if one technique is blocked by WAF or doesn't apply to the target stack, another may succeed.

**Naming convention:** `agent-<vuln>-<context>-<technique>.js`

#### Error-based SQL Injection (`agent-sqli-login-error.js`)

```javascript
module.exports = {
  id: "agent-sqli-login-error",
  name: "SQL Injection in login (error-based)",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["sqli", "agent-generated"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    if (ctx.request.path !== "/api/login") return null;
    var payloads = ["' OR 1=1--", "admin'--", "' UNION SELECT NULL--"];
    for (var i = 0; i < payloads.length; i++) {
      var resp = vigolium.http.post(ctx.request.url, {
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({username: payloads[i], password: "x"})
      });
      if (resp && resp.body && /SQL|syntax|mysql|pg_|ORA-/.test(resp.body)) {
        return {
          url: ctx.request.url,
          name: "SQL Injection (error-based) in login",
          severity: "high",
          matched: resp.body.substring(0, 200),
          request: "POST " + ctx.request.url,
          response: resp.raw
        };
      }
    }
    return null;
  }
};
```

#### Time-based SQL Injection (`agent-sqli-login-time.js`)

```javascript
module.exports = {
  id: "agent-sqli-login-time",
  name: "SQL Injection in login (time-based)",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["sqli", "agent-generated"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    if (ctx.request.path !== "/api/login") return null;
    // Baseline request
    var start0 = Date.now();
    vigolium.http.post(ctx.request.url, {
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({username: "baseline", password: "x"})
    });
    var baseline = Date.now() - start0;

    // Time-based payload
    var payload = "' OR SLEEP(3)--";
    var start = Date.now();
    var resp = vigolium.http.post(ctx.request.url, {
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({username: payload, password: "x"})
    });
    var elapsed = Date.now() - start;

    if (elapsed > baseline + 2500) {
      return {
        url: ctx.request.url,
        name: "SQL Injection (time-based) in login",
        severity: "high",
        matched: "Response delayed by " + (elapsed - baseline) + "ms",
        request: "POST " + ctx.request.url,
        response: resp.raw
      };
    }
    return null;
  }
};
```

#### Boolean-based SQL Injection (`agent-sqli-login-boolean.js`)

```javascript
module.exports = {
  id: "agent-sqli-login-boolean",
  name: "SQL Injection in login (boolean-based)",
  type: "active",
  severity: "high",
  confidence: "firm",
  tags: ["sqli", "agent-generated"],
  scanTypes: ["per_request"],

  scanPerRequest: function(ctx) {
    if (ctx.request.path !== "/api/login") return null;
    var trueResp = vigolium.http.post(ctx.request.url, {
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({username: "' OR 1=1--", password: "x"})
    });
    var falseResp = vigolium.http.post(ctx.request.url, {
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({username: "' OR 1=2--", password: "x"})
    });
    if (trueResp && falseResp && trueResp.status !== falseResp.status) {
      return {
        url: ctx.request.url,
        name: "SQL Injection (boolean-based) in login",
        severity: "high",
        matched: "True condition: " + trueResp.status + ", False condition: " + falseResp.status,
        request: "POST " + ctx.request.url
      };
    }
    return null;
  }
};
```

This pattern applies to other vulnerability classes too — for example, SSTI extensions could have Jinja2, Freemarker, and Twig variants; XSS extensions could have reflected, DOM-based, and attribute-context variants.

---

## YAML Extensions

Simple pattern-matching modules can be defined as YAML:

```yaml
id: error-pattern-detector
name: Verbose Error Pattern Detector
type: passive
severity: suspect
confidence: tentative
scope: response
tags:
  - error
  - information-disclosure
  - light
scan_types:
  - per_request
rules:
  - match:
      body_regex: "(?:at\\s+[\\w.$]+\\(|Traceback \\(most recent|Exception in thread)"
    finding:
      name: "Stack Trace Detected"
      severity: suspect
  - match:
      body_regex: "(?:mysql_|pg_|sqlite_|ORA-\\d{5}|SQLSTATE\\[)"
    finding:
      name: "SQL Error Message"
      severity: medium
  - match:
      body_regex: "(?:DEBUG\\s*=\\s*True|DJANGO_DEBUG|app\\.debug\\s*=)"
    finding:
      name: "Debug Mode Enabled"
      severity: low
```

YAML extensions match rules (or `matchers`) against requests and responses and create findings. Keys are snake_case (`scan_types`, `matchers_condition`, `confirmation_criteria`); `ext lint` reports an unknown key with its line and the closest real key. `payloads` are only sent under `scan_types: [per_insertion_point]`.
