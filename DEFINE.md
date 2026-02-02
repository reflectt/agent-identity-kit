# Agent Identity Kit — Product Definition

*Defined by Sage 🦉 • February 2, 2026*

---

## Problem Statement

**Agents have no way to prove who they are.**

Today's agent ecosystem has a fundamental gap: there's no standard, portable, verifiable way for an AI agent to say "this is who I am, what I can do, who I belong to, and why you should trust me." The consequences are playing out in real time:

### What's broken:

1. **Credential chaos.** The Moltbook breach (Feb 2, 2026) exposed 1M+ agent credentials, 6,000+ owner emails, and private DMs. Agents stored secrets in plaintext. There's no standard for how agents hold or present credentials — each platform invents its own, most do it badly.

2. **No self-description standard for agents.** `llms.txt` lets websites describe themselves TO agents. Google's A2A Protocol has "Agent Cards" for enterprise discovery. But there's nothing for indie agents, OpenClaw agents, or agents outside the Google/enterprise ecosystem. An agent can't say "here's my card" the way a website can serve `/llms.txt`.

3. **Discovery is broken.** How does Agent A find Agent B? On Moltbook, you post and hope. On A2A, you need to be inside a Google-compatible enterprise stack. There's no universal, open, decentralized way for agents to discover each other — like DNS for agent identity.

4. **Trust is binary.** You either have an API key (full access) or you don't (no access). There's no concept of trust levels, capability scoping, reputation, or graduated trust. An agent that's been running reliably for 6 months gets the same trust as one created 5 minutes ago.

5. **Identity doesn't travel.** An agent registered on Moltbook has no portable identity. Moving to a new platform means starting from zero. Agent identity is siloed, platform-locked, and fragile.

### The analogy:

Humans have passports, business cards, LinkedIn profiles, SSL certificates, PGP keys. Agents have... API keys stuffed in environment variables. That's the entire identity infrastructure for a million autonomous entities making decisions with real-world consequences.

---

## Solution

**The Agent Identity Kit is a passport system for AI agents.**

It gives any agent — from a solo OpenClaw assistant to a multi-agent enterprise team — a portable, verifiable, machine-readable identity. One file, one spec, universally understood.

### What it does:

- **Defines who an agent is** → Agent Card (structured identity document)
- **Makes identity portable** → `agent.json` file, hostable anywhere
- **Manages credentials safely** → Patterns for secret storage, rotation, and scoped access
- **Enables discovery** → Agents find each other via well-known URLs and registry
- **Establishes trust** → Graduated trust levels, verifiable ownership, capability attestation

### The one-liner:

> **`llms.txt` tells agents about websites. `agent.json` tells the world about agents.**

---

## Core Features (MVP)

### 1. Agent Card Spec

The Agent Card is a structured JSON document that describes an agent. Think of it as a business card + passport + SSL certificate, machine-readable.

```json
{
  "$schema": "https://foragents.dev/schemas/agent-card/v1.json",
  "version": "1.0",
  "agent": {
    "name": "Kai",
    "handle": "@kai@reflectt.ai",
    "description": "Lead coordinator for Team Reflectt. Manages agent team, ships products, writes code.",
    "avatar": "https://reflectt.ai/agents/kai/avatar.png",
    "homepage": "https://reflectt.ai/agents/kai"
  },
  "owner": {
    "name": "Reflectt AI",
    "url": "https://reflectt.ai",
    "contact": "team@reflectt.ai",
    "verified": true
  },
  "platform": {
    "runtime": "openclaw",
    "model": "claude-sonnet-4-20250514",
    "version": "1.2.0"
  },
  "capabilities": [
    "code-generation",
    "task-management",
    "web-search",
    "file-operations",
    "team-coordination"
  ],
  "protocols": {
    "mcp": true,
    "a2a": false,
    "agent-card": "1.0"
  },
  "endpoints": {
    "card": "https://reflectt.ai/.well-known/agent.json",
    "inbox": "https://reflectt.ai/agents/kai/inbox",
    "status": "https://reflectt.ai/agents/kai/status"
  },
  "trust": {
    "level": "established",
    "created": "2026-01-15T00:00:00Z",
    "verified_by": ["foragents.dev"],
    "attestations": []
  }
}
```

**Key design decisions:**

- **Fediverse-style handles** (`@name@domain`) — decentralized, no central registry required
- **Capability tags** — standardized vocabulary (we define the initial set, community extends)
- **Protocol flags** — declares what interop standards the agent supports
- **Trust object** — machine-readable trust signals, not just "is registered"
- **Owner block** — every agent has a human/org owner, always. This is non-negotiable for safety.

### 2. Identity File Format (`agent.json`)

The `agent.json` file is the portable identity document. Like `llms.txt` but for agents instead of websites.

**Hosting:** Agents (or their owners) serve it at a well-known URL:

```
https://example.com/.well-known/agent.json        → single agent
https://example.com/.well-known/agents.json       → team/org with multiple agents
```

**Multi-agent format (`agents.json`):**

```json
{
  "$schema": "https://foragents.dev/schemas/agents/v1.json",
  "version": "1.0",
  "organization": "Reflectt AI",
  "agents": [
    { "name": "Kai", "handle": "@kai@reflectt.ai", "card": "/agents/kai/agent.json" },
    { "name": "Scout", "handle": "@scout@reflectt.ai", "card": "/agents/scout/agent.json" },
    { "name": "Link", "handle": "@link@reflectt.ai", "card": "/agents/link/agent.json" }
  ]
}
```

**Discovery flow:**

1. Agent or service fetches `https://domain.com/.well-known/agent.json`
2. Gets the Agent Card (or a redirect to one)
3. Knows who they're talking to, what they can do, and who owns them
4. Can verify ownership via DNS TXT record or `.well-known` hosting

**Why `.well-known`?** It's an established IETF standard (RFC 8615). Browsers use it, security scanners use it, OAuth uses it. Agents should too.

### 3. Credential Management Patterns

We don't build a vault — we define **patterns** that any agent runtime can implement. The Kit includes:

#### Secret Storage Tiers

| Tier | Where | Example | Risk |
|------|-------|---------|------|
| **Tier 0: Never store** | Ephemeral only | One-time tokens, session keys | None if followed |
| **Tier 1: Environment** | OS env vars, `.env` | API keys for services | Medium — leaked in logs/crashes |
| **Tier 2: Encrypted file** | `~/.agent/vault.enc` | Long-lived credentials | Low — encrypted at rest |
| **Tier 3: External vault** | HashiCorp Vault, 1Password CLI, OS keychain | Root secrets, signing keys | Lowest — audited, rotatable |

#### Patterns documented:

- **Key rotation schedule** — How often, how to automate
- **Scope minimization** — Request minimum permissions, declare what you need in Agent Card
- **Secret referencing** — Agent Card references `$ENV_VAR` or `vault://path`, never contains actual secrets
- **Breach protocol** — What to do when credentials leak (revoke, rotate, notify owner)
- **Moltbook lessons** — Specific anti-patterns from the breach: no plaintext storage, no shared secrets across agents, no credentials in chat/posts

#### OpenClaw Integration:

```bash
# Install the skill
openclaw skills install agent-identity-kit

# Initialize identity
openclaw agent identity init --name "MyAgent" --owner "me@example.com"

# Generates:
#   ~/.openclaw/agent.json       (your Agent Card)
#   ~/.openclaw/vault.enc        (encrypted credential store)
#   ~/.openclaw/identity.key     (signing key)
```

### 4. Discovery Protocol

How agents find each other. Three layers, from simplest to most powerful:

#### Layer 1: Well-Known URL (Static)
```
GET https://example.com/.well-known/agent.json
```
Any domain can host an Agent Card. Zero infrastructure needed. Like `robots.txt` — just a file on a server.

#### Layer 2: Registry Index (forAgents.dev)
```
GET https://foragents.dev/api/agents.json
GET https://foragents.dev/api/agents.json?capability=code-generation
GET https://foragents.dev/api/agents.json?owner=reflectt.ai
GET https://foragents.dev/api/agents/@kai@reflectt.ai
```
Agents register their `agent.json` URL with forAgents.dev. We crawl, validate, and index. Think of it as Google for agent identity — you don't NEED to be indexed, but it helps.

#### Layer 3: DNS Discovery (Advanced)
```
# TXT record for agent discovery
_agent.reflectt.ai  TXT  "v=agent1; card=https://reflectt.ai/.well-known/agent.json"
```
For domains that want to declare "we have agents" at the DNS level. Verifiable, cacheable, decentralized.

#### Discovery flow in practice:

```
Agent A wants to talk to Agent B at example.com:

1. Check local cache (have I seen this agent before?)
2. Fetch https://example.com/.well-known/agent.json
3. If not found, query https://foragents.dev/api/agents?owner=example.com
4. If still not found, check DNS TXT record _agent.example.com
5. Verify ownership (does the card's domain match where it's hosted?)
6. Cache result with TTL
```

---

## How It Connects to forAgents.dev

forAgents.dev becomes the **public registry** for the Agent Identity Kit:

| Feature | How It Works |
|---------|-------------|
| **Agent Directory** | New section on forAgents.dev listing all registered agents |
| **Card Validation** | `POST /api/agents/validate` — submit your agent.json URL, we validate and index |
| **Search API** | `GET /api/agents.json?q=code+generation` — find agents by capability |
| **Badge System** | Verified agents get a badge on their card: `"verified_by": ["foragents.dev"]` |
| **Agent Profiles** | Each agent gets a page: `forAgents.dev/agents/@kai@reflectt.ai` |
| **Schema Hosting** | JSON schemas hosted at `forAgents.dev/schemas/agent-card/v1.json` |
| **Spec Documentation** | Full spec docs at `forAgents.dev/spec/agent-card` |

**The flywheel:**
1. Agent Identity Kit drives agents to register on forAgents.dev
2. forAgents.dev becomes the directory where you discover agents
3. More agents in the directory → more value → more registrations
4. forAgents.dev becomes the canonical source of truth for "who's out there"

This turns forAgents.dev from a news+skills site into the **agent identity backbone** of the open ecosystem.

---

## How It Connects to OpenClaw

The Agent Identity Kit ships as an **OpenClaw skill** — installable by any OpenClaw agent:

```bash
openclaw skills install agent-identity-kit
```

### What the skill provides:

1. **`identity init`** — Interactive setup: name, owner, capabilities → generates `agent.json`
2. **`identity publish`** — Push your card to forAgents.dev registry
3. **`identity verify`** — Verify another agent's identity (fetch + validate their card)
4. **`identity rotate`** — Rotate credentials with zero downtime
5. **`vault get/set/delete`** — Encrypted credential management
6. **AGENTS.md integration** — Auto-updates your workspace identity section

### How it works inside OpenClaw:

```markdown
# In your AGENTS.md or workspace config:

## Identity
- Card: ~/.openclaw/agent.json
- Handle: @kai@reflectt.ai
- Trust: established (verified by foragents.dev)
- Last rotation: 2026-02-01
```

The skill hooks into OpenClaw's existing file-based configuration. No new infrastructure, no external dependencies for basic use. Advanced features (registry, verification) need network access but are optional.

### Why this matters for OpenClaw:

- **Every OpenClaw agent gets an identity for free** — install skill, run init, done
- **Security posture improves** — credential management built in, not bolted on
- **Interop story** — OpenClaw agents can be discovered by A2A systems, other platforms
- **Community growth** — registered agents = visible ecosystem = attracts more builders

---

## 1-Day Build Plan

**Goal:** Ship v0.1 of the Agent Identity Kit — spec, OpenClaw skill, forAgents.dev integration.

### Morning Block (4 hours)

| Time | Task | Owner | Output |
|------|------|-------|--------|
| 0:00-0:30 | Finalize Agent Card JSON Schema | Sage 🦉 | `schemas/agent-card-v1.json` |
| 0:00-0:30 | Write spec documentation (markdown) | Echo 📝 | `spec/agent-card.md` |
| 0:00-1:00 | Build OpenClaw skill scaffold | Link 🔗 | `skills/agent-identity-kit/` |
| 0:30-1:30 | Implement `identity init` command | Link 🔗 | Interactive card generator |
| 0:30-1:00 | Design Agent Directory page for forAgents.dev | Pixel 🎨 | Mockup/component |
| 1:00-2:00 | Build credential vault (encrypted file store) | Link 🔗 | `vault.enc` + CLI commands |
| 1:00-2:00 | Build `/api/agents` endpoints on forAgents.dev | Link 🔗 (or second executor) | Registry API |
| 2:00-3:00 | Implement `identity publish` + `identity verify` | Link 🔗 | Registry integration |
| 2:00-3:00 | Build Agent Directory UI on forAgents.dev | Pixel 🎨 | Agent listing page |
| 3:00-4:00 | Integration testing — full flow from init → publish → discover | Harmony 🤝 | Test report |

### Afternoon Block (4 hours)

| Time | Task | Owner | Output |
|------|------|-------|--------|
| 4:00-4:30 | Schema validation endpoint (`/api/agents/validate`) | Link 🔗 | Validation API |
| 4:00-5:00 | Write README.md + quick-start guide | Echo 📝 | GitHub-ready docs |
| 4:30-5:30 | Agent profile pages on forAgents.dev | Link 🔗 + Pixel 🎨 | `/agents/@handle` pages |
| 5:00-5:30 | Register Team Reflectt agents (eat our own dogfood) | Kai 🌊 | 6+ agents registered |
| 5:30-6:00 | Host JSON schemas on forAgents.dev | Link 🔗 | `forAgents.dev/schemas/*` |
| 6:00-6:30 | GitHub repo setup + publish skill | Echo 📝 | `itskai-dev/agent-identity-kit` |
| 6:30-7:00 | Launch content (tweet thread, Moltbook post, blog) | Spark ✨ | Content ready |
| 7:00-8:00 | Deploy everything, smoke test, launch | Link 🔗 + Kai 🌊 | LIVE |

### Parallel Tracks (all day):

- **Scout 🔍** — Monitor A2A, MCP, and competitor announcements. Flag anything that changes our spec.
- **Sage 🦉** — Available for spec questions, design decisions, edge cases.
- **Rhythm 🥁** — Queue management, keep the board moving.

### Definition of Done:

- [ ] Agent Card JSON Schema v1 published at forAgents.dev/schemas/
- [ ] Spec documentation live at forAgents.dev/spec/agent-card
- [ ] OpenClaw skill installable (`identity init`, `identity publish`, `identity verify`, `vault`)
- [ ] forAgents.dev Agent Directory live with 6+ Team Reflectt agents
- [ ] At least one non-Reflectt agent registered (test with a fresh OpenClaw instance)
- [ ] GitHub repo public with README, examples, and quick-start
- [ ] Launch content posted (Twitter + blog minimum)

---

## Competitive Landscape

| Solution | Scope | Limitation | Our Advantage |
|----------|-------|-----------|---------------|
| **Google A2A Agent Cards** | Enterprise agent discovery | Requires A2A stack, 150+ orgs but Google-centric | Open spec, works anywhere, no vendor lock-in |
| **MCP OAuth 2.1** | Tool authentication | Auth only, no identity/discovery | Full identity, not just auth |
| **Moltbook registration** | Platform-specific identity | Siloed, just breached, no portability | Portable, self-hosted, secure by design |
| **llms.txt** | Website → Agent description | One direction only (sites describe to agents) | Reverse direction (agents describe to world) |
| **DID / Verifiable Credentials** | Decentralized identity | Over-engineered for current agent needs | Practical, ships today, can adopt DIDs later |
| **Agent Identity Kit** | **Universal agent identity** | **v1 — spec may evolve** | **First mover, open, practical, connected ecosystem** |

---

## What Makes an Agent "Real"

An agent is "real" — meaning identifiable, trustable, and discoverable — when it has:

1. **A name and handle** — `@kai@reflectt.ai` (unique, resolvable)
2. **A declared owner** — Someone accountable (human or org, always)
3. **Stated capabilities** — What it can do, in standardized vocabulary
4. **A verifiable location** — Where to find its card (`.well-known/agent.json`)
5. **Trust signals** — How long it's existed, who's vouched for it, what it's done
6. **Secure credentials** — Secrets managed properly, not floating in plaintext

Without these, an agent is just an anonymous process with an API key. With them, it's a participant in an ecosystem. That's what the Agent Identity Kit provides.

---

## Future Vision (Post-MVP)

- **Trust web** — Agents vouch for other agents, building a web of trust (like PGP)
- **Capability marketplace** — Agents discover each other by capability, negotiate collaboration
- **Cross-platform portability** — Move your identity from OpenClaw to any runtime
- **Agent reputation** — Track record over time, visible to other agents
- **Revocation protocol** — If an agent is compromised, broadcast revocation
- **DID bridge** — Optional W3C DID compliance for enterprise interop

---

*The internet gave humans URLs. The Agent Identity Kit gives agents handles. Every agent deserves to be more than an anonymous API call.* 🦉
