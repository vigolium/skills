# vigolium-scanner

Coding-agent skill for operating the [Vigolium](https://www.vigolium.com/) web
vulnerability scanner CLI. Teaches Claude Code, Codex, Cursor, Pi, or any
[agentskills.io](https://agentskills.io)-compatible agent how to pick the right
command, construct correct flags, triage findings, and confirm them.

## Install

The skill is embedded in the `vigolium` binary, so the installed copy always
matches your CLI version. Prefer this over copying files by hand:

```bash
vigolium skills install --agent claude --scope project   # --agent claude|codex|agents
vigolium skills                                          # list bundled skills
vigolium skills get --full                               # print SKILL.md + every reference
```

| `--agent` | `--scope project` | `--scope global` |
|-----------|-------------------|------------------|
| `claude` | `.claude/skills/` | `~/.claude/skills/` |
| `codex` \| `agents` | `.agents/skills/` | `~/.agents/skills/` |

Once installed it auto-triggers on mentions of scanning, vigolium, DAST, or
vulnerability testing. In Claude Code you can also invoke it explicitly as
`/vigolium-scanner`.

Alternatives (not version-matched): `bunx skills add vigolium/skills --skill
vigolium-scanner --agent <name> --yes`, or clone
[vigolium/skills](https://github.com/vigolium/skills) and copy the folder into
`~/.claude/skills/` or `~/.agents/skills/`.

## Layout

```
vigolium-scanner/
├── SKILL.md                      # always loaded: router, mental model, invariants
└── references/                   # loaded on demand, one hop from SKILL.md
    ├── agent-loop.md             # ★ driving vigolium from an agent: -j contracts,
    │                             #   triage, replay, exports, exit codes
    ├── scanning.md               # scan, scan-url, scan-request, run, phases, strategies
    ├── fuzzing.md                # vigolium fuzz: positions, attack modes, anomaly scoring
    ├── burp.md                   # Burp bridge, Repeater/Organizer/Site map, --send-via-burp
    ├── agent-modes.md            # agent query/autopilot/swarm/audit/olium/triage/session
    ├── auth.md                   # --auth-file / --auth, YAML format, extract rules
    ├── data.md                   # db, finding, traffic, module, ext, js, config, export
    ├── server.md                 # vigolium server: REST API, recording/MITM proxy, live mirror
    ├── ingest.md                 # vigolium ingest: local/remote, per-format input examples
    ├── extensions.md             # writing JS scanner modules against vigolium.*
    └── flags.generated.md        # GENERATED from the cobra tree — grep it by flag name
```

`SKILL.md` is the only file always in context; it routes to at most one reference
per task. If you are building an agent integration, start with
`references/agent-loop.md` — it documents the `scope → scan → read → confirm →
hand off` loop, the two JSON contracts, and the token-bounding flags.

## Maintaining

`references/flags.generated.md` is generated — never edit it by hand:

```bash
make skill-flags          # or: vigolium skills gen-flags
```

Re-run it whenever a flag is added, renamed, or given a shorthand; the file is
committed so the embedded bundle stays self-contained.

Everything else is hand-written prose. Two rules keep it useful:

1. **Keep `SKILL.md` under ~500 lines.** It is loaded in full every time the
   skill triggers, so anything that belongs to one topic belongs in that topic's
   reference instead.
2. **Don't restate flags.** If `vigolium <cmd> -h` answers it, link to the
   generated reference rather than copying rows that will drift.

## Docs

- Install & usage guide: [`docs/agentic-scan/using-vigolium-in-agent.md`](../../docs/agentic-scan/using-vigolium-in-agent.md)
- Driving vigolium from an agent: [`docs/coding-agent.md`](../../docs/coding-agent.md)
- Full documentation: [docs.vigolium.com](https://docs.vigolium.com/)
- Cheat sheet: [docs.vigolium.com/getting-started/cheat-sheet](https://docs.vigolium.com/getting-started/cheat-sheet)
- Skills repository: [github.com/vigolium/skills](https://github.com/vigolium/skills)
