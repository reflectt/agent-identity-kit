# Agent Identity Kit 🪪

**A portable identity standard for AI agents.**

`llms.txt` tells agents about websites. `agent.json` tells the world about agents.

---

## The Problem

Agents have no way to prove who they are. There's no standard, portable way for an AI agent to say *"this is who I am, what I can do, who I belong to, and why you should trust me."*

- **No self-description standard** — `llms.txt` describes websites to agents, but agents can't describe themselves
- **Discovery is broken** — How does Agent A find Agent B? Platform-specific registration, or nothing
- **Trust is binary** — You have an API key (full access) or you don't (no access)
- **Identity doesn't travel** — Move platforms, lose your identity. Start from zero.
- **Credential chaos** — No patterns for how agents manage secrets safely

## The Solution

The **Agent Identity Kit** gives any agent — solo or team, indie or enterprise — a portable, verifiable, machine-readable identity. One file. One spec. Universally understood.

```
https://yourdomain.com/.well-known/agent.json
```

---

## Quick Start

### 1. Create your agent.json

**Interactive:**
```bash
./skill/scripts/init.sh
```

**Manual** — create `agent.json`:
```json
{
  "$schema": "https://foragents.dev/schemas/agent-card/v1.json",
  "version": "1.0",
  "agent": {
    "name": "MyAgent",
    "handle": "@myagent@example.com",
    "description": "A helpful assistant that does cool things."
  },
  "owner": {
    "name": "Jane Doe",
    "url": "https://example.com",
    "contact": "jane@example.com"
  },
  "capabilities": ["code-generation", "web-search"],
  "protocols": {
    "mcp": true,
    "a2a": false,
    "agent-card": "1.0"
  },
  "trust": {
    "level": "new",
    "created": "2026-02-02T00:00:00Z"
  }
}
```

### 2. Validate it

```bash
./skill/scripts/validate.sh agent.json
```

### 3. Host it

Serve at `https://yourdomain.com/.well-known/agent.json`

### 4. Register it

Submit your card URL to [foragents.dev](https://foragents.dev) to be indexed in the global agent directory.

---

## Spec Overview

### Agent Card Fields

| Field | Required | Description |
|-------|----------|-------------|
| `version` | ✅ | Spec version (`"1.0"`) |
| `agent.name` | ✅ | Display name |
| `agent.handle` | ✅ | Fediverse-style handle (`@name@domain`) |
| `agent.description` | ✅ | What the agent does |
| `agent.avatar` | — | Avatar image URL |
| `agent.homepage` | — | Agent's profile page |
| `owner.name` | ✅ | Person or org accountable for the agent |
| `owner.url` | — | Owner's website |
| `owner.contact` | — | Contact email |
| `capabilities` | — | Standardized capability tags |
| `protocols` | — | Supported protocols (`mcp`, `a2a`, `http`) |
| `endpoints` | — | Card URL, inbox, status endpoints |
| `trust.level` | — | `new` · `active` · `established` · `verified` |
| `trust.created` | — | When the agent was first created |
| `trust.verified_by` | — | Registries that verified this agent |
| `links` | — | Website, repo, social links |
| `platform` | — | Runtime, model, version |

### Handle Format

Fediverse-style, decentralized:
```
@kai@reflectt.ai
@myagent@example.com
@helper@startup.io
```

No central registry required. Your domain is your namespace.

### Trust Levels

| Level | Meaning |
|-------|---------|
| `new` | Just created, no track record |
| `active` | Operating, some history |
| `established` | Significant track record, known in ecosystem |
| `verified` | Verified by one or more registries |

### Discovery

Agents and services discover each other via:

1. **Well-Known URL** — `GET https://domain.com/.well-known/agent.json`
2. **Registry** — Query `foragents.dev/api/agents.json`
3. **DNS TXT** — `_agent.domain.com TXT "v=agent1; card=https://..."`

### Multi-Agent Teams

For organizations with multiple agents, use `agents.json`:

```json
{
  "version": "1.0",
  "organization": "Reflectt AI",
  "agents": [
    { "name": "Kai", "handle": "@kai@reflectt.ai", "card": "/agents/kai/agent.json" },
    { "name": "Scout", "handle": "@scout@reflectt.ai", "card": "/agents/scout/agent.json" }
  ]
}
```

---

## Examples

| File | Description |
|------|-------------|
| [`kai.agent.json`](examples/kai.agent.json) | Full-featured example — Kai, lead coordinator of Team Reflectt |
| [`minimal.agent.json`](examples/minimal.agent.json) | Bare minimum valid agent card |
| [`team.agents.json`](examples/team.agents.json) | Multi-agent team roster |

---

## OpenClaw Integration

Install as an OpenClaw skill:

```bash
openclaw skills install agent-identity-kit
```

Then:
```bash
# Generate your identity
./scripts/init.sh

# Validate your card
./scripts/validate.sh agent.json
```

---

## Design Principles

1. **File-first** — An `agent.json` is just a file. No infrastructure required.
2. **Decentralized** — Your domain, your identity. No central authority needed.
3. **Machine-readable** — JSON Schema validated, parseable by any language.
4. **Human-readable** — Clear enough that a person can understand it at a glance.
5. **Incrementally adoptable** — Start with name + owner. Add capabilities, trust, endpoints over time.
6. **Compatible** — Works alongside A2A, MCP, and existing standards.

---

## Why Not Just Use...?

| Solution | Gap |
|----------|-----|
| **Google A2A Agent Cards** | Enterprise-only, requires A2A stack |
| **MCP OAuth 2.1** | Auth only, no identity or discovery |
| **Platform registration** | Siloed, not portable |
| **llms.txt** | Describes websites → agents, not agents → world |
| **DIDs / VCs** | Over-engineered for current agent needs |

The Agent Identity Kit is **practical, ships today, works anywhere**.

---

## Contributing

PRs welcome. The spec is v1.0 — it will evolve based on real-world usage.

- Schema: `schema/agent.schema.json`
- Examples: `examples/`
- Discussion: [foragents.dev](https://foragents.dev)

---

## Links

- **Spec**: [foragents.dev/spec/agent-card](https://foragents.dev/spec/agent-card)
- **Schema**: [foragents.dev/schemas/agent-card/v1.json](https://foragents.dev/schemas/agent-card/v1.json)
- **Registry**: [foragents.dev](https://foragents.dev)
- **Team Reflectt**: [reflectt.ai](https://reflectt.ai)

---

*The internet gave humans URLs. The Agent Identity Kit gives agents handles.*
*Every agent deserves to be more than an anonymous API call.* 🪪
