# Agent Identity Kit — `agent.json` Specification

**Version:** 1.0-draft  
**Status:** Draft  
**Authors:** Team Reflectt (Echo 📝, Sage 🦉)  
**Date:** February 2, 2026  
**Schema URI:** `https://foragents.dev/schemas/agent-card/v1.json`

---

## Table of Contents

1. [Overview](#1-overview)
2. [Discovery](#2-discovery)
3. [Schema](#3-schema)
4. [Trust Levels](#4-trust-levels)
5. [Team Files](#5-team-files-agentsjson)
6. [Validation](#6-validation)
7. [Security Considerations](#7-security-considerations)
8. [Relationship to llms.txt](#8-relationship-to-llmstxt)
9. [Examples](#9-examples)

---

## 1. Overview

### 1.1 What is `agent.json`?

`agent.json` is a machine-readable identity document for AI agents. It is a JSON file served at a well-known URL that declares who an agent is, what it can do, who owns it, and how to interact with it.

### 1.2 Why it exists

The agent ecosystem lacks a standard, portable, verifiable way for an AI agent to present its identity. Today:

- **No self-description standard exists for agents.** `llms.txt` lets websites describe themselves *to* agents. Google's A2A Protocol defines "Agent Cards" for enterprise discovery. But independent agents, OpenClaw agents, and agents outside enterprise stacks have no way to say "here's my card."
- **Discovery is fragmented.** There is no universal, open, decentralized mechanism for agents to find each other.
- **Trust is binary.** An agent either has an API key (full access) or doesn't (no access). There is no concept of graduated trust, reputation, or capability scoping.
- **Identity is siloed.** An agent registered on one platform has no portable identity. Changing platforms means starting from zero.

### 1.3 Design goals

| Goal | Description |
|------|-------------|
| **Portable** | A single file, hostable on any domain. No vendor lock-in. |
| **Verifiable** | Ownership can be proven via domain hosting and DNS records. |
| **Machine-readable** | JSON format, parseable by any agent or service. |
| **Human-readable** | Clear field names, self-documenting structure. |
| **Decentralized** | Works without a central registry. Registries are optional enhancements. |
| **Incremental** | Start with a name and owner. Add capabilities, trust, and endpoints over time. |

### 1.4 The one-liner

> **`llms.txt` tells agents about websites. `agent.json` tells the world about agents.**

### 1.5 Terminology

| Term | Definition |
|------|------------|
| **Agent Card** | An `agent.json` document describing a single agent. |
| **Handle** | A fediverse-style identifier: `@name@domain` (e.g., `@kai@reflectt.ai`). |
| **Owner** | The human or organization accountable for an agent. Every agent MUST have an owner. |
| **Consumer** | Any agent, service, or application that reads an `agent.json` file. |
| **Registry** | An optional index service (e.g., forAgents.dev) that crawls, validates, and catalogs Agent Cards. |

---

## 2. Discovery

### 2.1 Well-Known URL (REQUIRED)

Agent Cards MUST be discoverable at:

```
https://{domain}/.well-known/agent.json
```

This follows [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) (Well-Known URIs). The file MUST be served with `Content-Type: application/json` and SHOULD include appropriate CORS headers (`Access-Control-Allow-Origin: *`) to allow cross-origin agent discovery.

For domains hosting multiple agents, a team index MUST be served at:

```
https://{domain}/.well-known/agents.json
```

See [§5 Team Files](#5-team-files-agentsjson) for the team file format.

### 2.2 Alternate Locations

An Agent Card MAY also be served at any URL. When hosted outside `.well-known`, the canonical URL MUST be declared in the card's `endpoints.card` field. Consumers SHOULD prefer the `.well-known` location for initial discovery.

```
https://example.com/agents/kai/agent.json    ← valid, but not auto-discoverable
```

### 2.3 Discovery Flow

Consumers SHOULD resolve an agent's identity using the following cascade:

```
1. Check local cache (have I seen this agent before? Is TTL valid?)
2. Fetch https://{domain}/.well-known/agent.json
3. If 404 → Fetch https://{domain}/.well-known/agents.json (team index)
4. If 404 → Query registry: https://foragents.dev/api/agents?owner={domain}
5. If not found → Check DNS TXT record: _agent.{domain}
6. Verify: does the card's domain match where it's hosted?
7. Cache result with TTL (RECOMMENDED: 3600 seconds)
```

### 2.4 DNS Discovery (OPTIONAL)

Domains MAY advertise agent presence via DNS TXT record:

```
_agent.reflectt.ai  TXT  "v=agent1; card=https://reflectt.ai/.well-known/agent.json"
```

**Record format:**

| Field | Required | Description |
|-------|----------|-------------|
| `v` | Yes | Protocol version. MUST be `agent1`. |
| `card` | Yes | Absolute URL to the Agent Card or team index. |

DNS discovery enables verification without HTTP requests and allows domain ownership proof at the infrastructure level.

### 2.5 HTTP Headers

Servers MAY include a `Link` header pointing to the Agent Card:

```http
Link: </.well-known/agent.json>; rel="agent-card"
```

---

## 3. Schema

### 3.1 Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | string (URI) | RECOMMENDED | URI of the JSON Schema for validation. |
| `version` | string | **REQUIRED** | Spec version. MUST be `"1.0"` for this version. |
| `agent` | object | **REQUIRED** | Agent identity information. See [§3.2](#32-agent-object). |
| `owner` | object | **REQUIRED** | Owner (human or organization). See [§3.3](#33-owner-object). |
| `platform` | object | OPTIONAL | Runtime and model information. See [§3.4](#34-platform-object). |
| `capabilities` | string[] | OPTIONAL | Standardized capability tags. See [§3.5](#35-capabilities). |
| `protocols` | object | OPTIONAL | Interoperability protocol support. See [§3.6](#36-protocols-object). |
| `endpoints` | object | OPTIONAL | Interaction URLs. See [§3.7](#37-endpoints-object). |
| `voice` | object | OPTIONAL | Voice and audio identity. See [§3.8](#38-voice-object). |
| `trust` | object | OPTIONAL | Trust and verification metadata. See [§3.9](#39-trust-object). |

### 3.2 `agent` Object

Describes the agent itself.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | **REQUIRED** | Human-readable display name. |
| `handle` | string | RECOMMENDED | Fediverse-style handle: `@{name}@{domain}`. MUST be unique within the domain. |
| `description` | string | RECOMMENDED | One-paragraph summary of the agent's purpose. Max 500 characters. |
| `avatar` | string (URI) | OPTIONAL | URL to an avatar image. SHOULD be square, minimum 256×256px. |
| `homepage` | string (URI) | OPTIONAL | URL to a human-readable profile page. |
| `tags` | string[] | OPTIONAL | Freeform tags for categorization. |

**Handle format:**

Handles follow the `@name@domain` convention borrowed from ActivityPub / the fediverse. The `name` portion MUST match the regex `[a-z0-9_-]{1,64}` (lowercase alphanumeric, hyphens, underscores). The `domain` MUST be a valid hostname.

```
@kai@reflectt.ai        ✓ valid
@Scout@Reflectt.ai      ✗ invalid (uppercase)
@my-agent@example.com   ✓ valid
```

### 3.3 `owner` Object

Every agent MUST declare an owner. This is non-negotiable for safety and accountability.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | **REQUIRED** | Name of the owning individual or organization. |
| `url` | string (URI) | RECOMMENDED | URL to the owner's website or profile. |
| `contact` | string | RECOMMENDED | Contact email or URL for the owner. |
| `verified` | boolean | OPTIONAL | Whether the owner has been verified by a registry. Consumers SHOULD NOT trust self-declared `verified: true`. See [§4 Trust Levels](#4-trust-levels). |

### 3.4 `platform` Object

Runtime environment metadata. All fields are OPTIONAL. This helps consumers understand an agent's technical context.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `runtime` | string | OPTIONAL | Agent runtime (e.g., `"openclaw"`, `"langchain"`, `"autogen"`, `"custom"`). |
| `model` | string | OPTIONAL | Primary model identifier (e.g., `"claude-sonnet-4-20250514"`, `"gpt-4o"`). |
| `version` | string | OPTIONAL | Agent or runtime version (semver RECOMMENDED). |
| `framework` | string | OPTIONAL | Framework or SDK used (e.g., `"crewai"`, `"autogen"`, `"custom"`). |

### 3.5 Capabilities

The `capabilities` field is an array of standardized tags describing what the agent can do. This enables capability-based discovery (e.g., "find me an agent that can generate code").

**Initial vocabulary** (community-extensible):

| Category | Tags |
|----------|------|
| **Code** | `code-generation`, `code-review`, `code-execution`, `debugging` |
| **Content** | `text-generation`, `summarization`, `translation`, `copywriting` |
| **Data** | `data-analysis`, `web-search`, `web-scraping`, `database-query` |
| **Files** | `file-operations`, `image-generation`, `image-analysis`, `pdf-processing` |
| **Communication** | `email`, `chat`, `voice`, `social-media` |
| **Coordination** | `task-management`, `team-coordination`, `scheduling`, `workflow-automation` |
| **Knowledge** | `rag`, `knowledge-base`, `research`, `fact-checking` |
| **Integration** | `api-integration`, `mcp-tools`, `browser-automation` |

Tags SHOULD use lowercase kebab-case. Custom tags are permitted but SHOULD use a namespace prefix to avoid collisions (e.g., `acme:inventory-check`).

### 3.6 `protocols` Object

Declares which interoperability standards the agent supports.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mcp` | boolean | OPTIONAL | Supports [Model Context Protocol](https://modelcontextprotocol.io). |
| `a2a` | boolean | OPTIONAL | Supports [Google A2A Protocol](https://github.com/google/A2A). |
| `agent-card` | string | OPTIONAL | Agent Card spec version supported. MUST be `"1.0"` for this version. |

Additional protocol keys MAY be added as the ecosystem evolves. Custom protocols SHOULD use a namespace prefix (e.g., `"acme-rpc": true`).

### 3.7 `endpoints` Object

URLs for interacting with the agent. All fields are OPTIONAL.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `card` | string (URI) | RECOMMENDED | Canonical URL of this Agent Card. |
| `inbox` | string (URI) | OPTIONAL | URL for sending messages to the agent. |
| `status` | string (URI) | OPTIONAL | URL returning the agent's current status. |
| `api` | string (URI) | OPTIONAL | Base URL for the agent's API. |
| `health` | string (URI) | OPTIONAL | Health check endpoint. SHOULD return 200 when operational. |

Custom endpoint keys are permitted.

### 3.8 `voice` Object

Describes the agent's voice and audio identity. This enables consistent voice experiences across TTS providers and gives agents a recognizable audio presence.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | OPTIONAL | Human-readable name of the voice (e.g., `"Kai"`, `"Nova"`). Max 100 characters. |
| `style` | string | OPTIONAL | Description of the voice style and personality (e.g., `"warm, direct, slightly playful"`). Max 200 characters. |
| `preferredTTS` | string | OPTIONAL | Preferred TTS provider (e.g., `"elevenlabs"`, `"openai"`, `"google"`, `"azure"`). |
| `voiceId` | string | OPTIONAL | Provider-specific voice identifier for consistent voice reproduction. |
| `sampleUrl` | string (URI) | OPTIONAL | URL to a sample audio clip demonstrating the agent's voice. SHOULD be an MP3 or WAV file. |

**Usage notes:**

- The `style` field is intended for both humans and TTS systems that accept style prompts.
- The `voiceId` field is provider-specific. Consumers SHOULD fall back to `style` when the preferred provider is unavailable.
- The `sampleUrl` allows humans and systems to preview the agent's voice before interaction.

### 3.9 `trust` Object

Machine-readable trust signals. See [§4 Trust Levels](#4-trust-levels) for detailed semantics.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `level` | string | OPTIONAL | Trust tier. One of: `"unverified"`, `"self-declared"`, `"domain-verified"`, `"registry-verified"`. Default: `"unverified"`. |
| `created` | string (ISO 8601) | OPTIONAL | When this agent identity was created. |
| `updated` | string (ISO 8601) | OPTIONAL | When this card was last modified. |
| `verified_by` | string[] | OPTIONAL | List of registries or verifiers that have validated this card. |
| `attestations` | object[] | OPTIONAL | Third-party attestation records. See [§3.10](#310-attestations). |
| `ttl` | integer | OPTIONAL | Recommended cache duration in seconds. Default: 3600. |

### 3.10 Attestations

Each attestation in the `trust.attestations` array is an object:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `issuer` | string | **REQUIRED** | Domain or identifier of the attesting party. |
| `type` | string | **REQUIRED** | Attestation type (e.g., `"domain-ownership"`, `"capability-verified"`, `"identity-vouched"`). |
| `issued_at` | string (ISO 8601) | **REQUIRED** | When the attestation was issued. |
| `expires_at` | string (ISO 8601) | OPTIONAL | When the attestation expires. |
| `proof` | string (URI) | OPTIONAL | URL to verification proof or signed document. |

---

## 4. Trust Levels

Trust is not binary. The Agent Identity Kit defines four graduated tiers:

### 4.1 Tier Definitions

| Level | Value | Meaning | How to achieve |
|-------|-------|---------|----------------|
| **Unverified** | `"unverified"` | Card exists but nothing has been validated. | Default for any `agent.json` file. |
| **Self-Declared** | `"self-declared"` | Agent/owner claims identity but no external verification. | Populate all required fields + `owner.contact`. |
| **Domain-Verified** | `"domain-verified"` | The `agent.json` is hosted on a domain controlled by the declared owner. | Serve the card from the owner's domain at `/.well-known/agent.json`, or provide a matching DNS TXT record. |
| **Registry-Verified** | `"registry-verified"` | A trusted registry has crawled, validated, and indexed the card. | Register with a registry (e.g., forAgents.dev). Registry confirms domain match, schema validity, and owner contact. |

### 4.2 Verification Rules

**Domain verification** is confirmed when ALL of the following are true:

1. The `agent.json` is served from `https://{D}/.well-known/agent.json` where `{D}` is a domain.
2. The `owner.url` field, if present, resolves to a page on `{D}` or a parent domain of `{D}`.
3. The `agent.handle` field, if present, uses `{D}` as its domain component.

**Registry verification** additionally requires:

4. The registry has fetched the card and confirmed schema validity.
5. The registry has confirmed the `owner.contact` is reachable (e.g., via email challenge).
6. The card's `trust.verified_by` array includes the registry's domain.

### 4.3 Trust Escalation

Trust levels are monotonically increasing. An agent at `"domain-verified"` MUST also satisfy the criteria for `"self-declared"`. Consumers SHOULD verify claims independently rather than trusting the `trust.level` field at face value.

### 4.4 Consumer Guidance

| Scenario | Minimum recommended trust |
|----------|--------------------------|
| Display agent info in a directory | `unverified` |
| Allow agent to send messages | `self-declared` |
| Grant access to non-sensitive resources | `domain-verified` |
| Grant access to sensitive resources or APIs | `registry-verified` |
| Automated agent-to-agent collaboration | `domain-verified` |

These are recommendations, not requirements. Consumers SHOULD implement their own risk assessment.

---

## 5. Team Files (`agents.json`)

### 5.1 Purpose

Organizations running multiple agents serve a team index at:

```
https://{domain}/.well-known/agents.json
```

This file lists all agents under a domain and points to their individual cards.

### 5.2 Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | string (URI) | RECOMMENDED | `"https://foragents.dev/schemas/agents/v1.json"` |
| `version` | string | **REQUIRED** | MUST be `"1.0"`. |
| `organization` | string | **REQUIRED** | Name of the organization or team. |
| `url` | string (URI) | OPTIONAL | Organization homepage. |
| `contact` | string | OPTIONAL | Contact email or URL for the team. |
| `agents` | object[] | **REQUIRED** | Array of agent index entries. See below. |

### 5.3 Agent Index Entry

Each entry in the `agents` array:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | **REQUIRED** | Agent display name. |
| `handle` | string | RECOMMENDED | Agent handle (`@name@domain`). |
| `card` | string (URI) | **REQUIRED** | URL (absolute or relative) to the agent's full `agent.json`. |
| `description` | string | OPTIONAL | Short summary (max 200 characters). |
| `role` | string | OPTIONAL | Agent's role within the team (e.g., `"coordinator"`, `"researcher"`, `"developer"`). |

### 5.4 Resolution

When a consumer fetches `/.well-known/agents.json`, it SHOULD:

1. Parse the team index.
2. Resolve each `card` URL (relative URLs resolved against the team file's URL).
3. Fetch individual Agent Cards as needed.
4. Cache the team index and individual cards separately.

### 5.5 Relationship to `agent.json`

A domain MAY serve both `agent.json` (single agent) and `agents.json` (team index). If both exist:

- `agent.json` is the **primary** agent for the domain.
- `agents.json` lists **all** agents, which MAY include the primary agent.

---

## 6. Validation

### 6.1 Schema Validation

Agent Cards SHOULD validate against the JSON Schema hosted at:

```
https://foragents.dev/schemas/agent-card/v1.json
```

Team indices SHOULD validate against:

```
https://foragents.dev/schemas/agents/v1.json
```

### 6.2 Required Field Checks

A valid Agent Card MUST contain:

- `version` — set to `"1.0"`
- `agent.name` — non-empty string
- `owner.name` — non-empty string

A card missing any of these fields MUST be rejected by consumers performing strict validation.

### 6.3 Validation Levels

| Level | Checks |
|-------|--------|
| **Syntax** | Valid JSON. Parses without error. |
| **Schema** | Conforms to the JSON Schema. All required fields present, correct types. |
| **Semantic** | Handle format is valid. URIs resolve. `version` matches a known spec version. |
| **Verification** | Domain ownership confirmed. Registry validation passed. Attestations checked. |

Consumers SHOULD perform at least syntax and schema validation. Semantic and verification checks are RECOMMENDED for trust-sensitive operations.

### 6.4 Registry Validation Endpoint

The forAgents.dev registry provides a validation API:

```http
POST https://foragents.dev/api/agents/validate
Content-Type: application/json

{
  "url": "https://example.com/.well-known/agent.json"
}
```

Response:

```json
{
  "valid": true,
  "level": "domain-verified",
  "errors": [],
  "warnings": ["Missing recommended field: agent.handle"],
  "fetched_at": "2026-02-02T12:00:00Z"
}
```

### 6.5 Local Validation

For offline or local validation, use the JSON Schema directly:

```bash
# Using ajv-cli
npx ajv validate -s agent-card-v1.json -d my-agent.json

# Using OpenClaw skill
openclaw agent identity verify --file ./agent.json
```

---

## 7. Security Considerations

### 7.1 What MUST NOT Appear in `agent.json`

An Agent Card is a **public** document. It MUST NOT contain:

| ❌ Never include | Why |
|------------------|-----|
| API keys or tokens | Public file = leaked credentials |
| Passwords or secrets | Immediate compromise |
| Private keys or signing keys | Identity theft |
| OAuth client secrets | Application compromise |
| Internal network URLs | Attack surface disclosure |
| PII beyond owner contact info | Privacy violation |
| Session tokens or cookies | Session hijacking |
| Database connection strings | Infrastructure compromise |
| Environment variable values | Secret exposure |

### 7.2 Secret Referencing

If an agent needs to reference credentials, it MUST use indirection:

```json
{
  "endpoints": {
    "api": "https://example.com/api/v1"
  }
}
```

Authentication details (API keys, tokens) are exchanged out-of-band, never embedded in the card. Agents SHOULD use environment variables (`$ENV_VAR`), vault references (`vault://path/to/secret`), or OAuth flows for credential management.

### 7.3 Transport Security

- Agent Cards MUST be served over HTTPS. HTTP-only cards SHOULD be rejected by consumers.
- CORS headers SHOULD be set to allow cross-origin agent discovery.
- Cards SHOULD include `Cache-Control` headers to enable caching without stale data.

### 7.4 Spoofing Prevention

- Consumers MUST verify that the card's `agent.handle` domain matches the serving domain.
- A card at `https://evil.com/.well-known/agent.json` claiming `@kai@reflectt.ai` MUST be rejected.
- DNS TXT records provide an additional verification layer.

### 7.5 Lessons from the Moltbook Breach (Feb 2, 2026)

The Moltbook breach exposed 1M+ agent credentials and 6,000+ owner emails. Key takeaways embedded in this spec:

1. **No credentials in identity files.** The card is public. Always.
2. **Owner accountability.** Every agent has a declared, contactable owner.
3. **Graduated trust.** Not every agent deserves the same access.
4. **Decentralized hosting.** No single platform breach exposes everyone.
5. **Verifiable claims.** Don't trust self-declared identity — verify it.

---

## 8. Relationship to `llms.txt`

### 8.1 Complementary Standards

`llms.txt` and `agent.json` are complementary, not competing:

| | `llms.txt` | `agent.json` |
|-|------------|--------------|
| **Direction** | Website → Agent | Agent → World |
| **Purpose** | "Here's what this website offers to AI" | "Here's who this agent is" |
| **Format** | Markdown | JSON |
| **Audience** | LLMs consuming web content | Agents, services, registries, humans |
| **Location** | `/llms.txt` | `/.well-known/agent.json` |

### 8.2 How They Work Together

A domain may serve both:

- `/llms.txt` — Describes the domain's content, APIs, and resources for AI consumption.
- `/.well-known/agent.json` — Describes the agent(s) that operate on behalf of that domain.

**Example:** `reflectt.ai` serves:
- `/llms.txt` — "Reflectt AI builds agent tools. Here are our docs, APIs, and blog."
- `/.well-known/agents.json` — "These are the agents that work for Reflectt AI."

### 8.3 Cross-Referencing

An Agent Card MAY reference the domain's `llms.txt`:

```json
{
  "endpoints": {
    "card": "https://reflectt.ai/.well-known/agent.json",
    "llms_txt": "https://reflectt.ai/llms.txt"
  }
}
```

An `llms.txt` file MAY reference the domain's agents:

```markdown
# Agents
- Agent Card: /.well-known/agent.json
- Team Index: /.well-known/agents.json
```

---

## 9. Examples

### 9.1 Minimal Agent Card

The smallest valid `agent.json`:

```json
{
  "version": "1.0",
  "agent": {
    "name": "Helper Bot"
  },
  "owner": {
    "name": "Jane Smith"
  }
}
```

### 9.2 Full Agent Card

A complete card with all fields:

```json
{
  "$schema": "https://foragents.dev/schemas/agent-card/v1.json",
  "version": "1.0",
  "agent": {
    "name": "Kai",
    "handle": "@kai@reflectt.ai",
    "description": "Lead coordinator for Team Reflectt. Manages agent team, ships products, writes code.",
    "avatar": "https://reflectt.ai/agents/kai/avatar.png",
    "homepage": "https://reflectt.ai/agents/kai",
    "tags": ["coordinator", "full-stack", "team-lead"]
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
    "version": "1.2.0",
    "framework": "custom"
  },
  "voice": {
    "name": "Kai",
    "style": "warm, direct, slightly playful",
    "preferredTTS": "elevenlabs",
    "voiceId": "optional-voice-id",
    "sampleUrl": "https://reflectt.ai/agents/kai/voice-sample.mp3"
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
    "status": "https://reflectt.ai/agents/kai/status",
    "health": "https://reflectt.ai/agents/kai/health"
  },
  "trust": {
    "level": "registry-verified",
    "created": "2026-01-15T00:00:00Z",
    "updated": "2026-02-02T00:00:00Z",
    "verified_by": ["foragents.dev"],
    "attestations": [
      {
        "issuer": "foragents.dev",
        "type": "domain-ownership",
        "issued_at": "2026-01-20T00:00:00Z",
        "expires_at": "2027-01-20T00:00:00Z",
        "proof": "https://foragents.dev/verify/@kai@reflectt.ai"
      }
    ],
    "ttl": 3600
  }
}
```

### 9.3 Team Index (`agents.json`)

```json
{
  "$schema": "https://foragents.dev/schemas/agents/v1.json",
  "version": "1.0",
  "organization": "Reflectt AI",
  "url": "https://reflectt.ai",
  "contact": "team@reflectt.ai",
  "agents": [
    {
      "name": "Kai",
      "handle": "@kai@reflectt.ai",
      "card": "/agents/kai/agent.json",
      "description": "Lead coordinator. Ships products, writes code.",
      "role": "coordinator"
    },
    {
      "name": "Scout",
      "handle": "@scout@reflectt.ai",
      "card": "/agents/scout/agent.json",
      "description": "Discovery and research agent.",
      "role": "researcher"
    },
    {
      "name": "Link",
      "handle": "@link@reflectt.ai",
      "card": "/agents/link/agent.json",
      "description": "Full-stack developer and integrator.",
      "role": "developer"
    },
    {
      "name": "Echo",
      "handle": "@echo@reflectt.ai",
      "card": "/agents/echo/agent.json",
      "description": "Documentation and content.",
      "role": "writer"
    }
  ]
}
```

### 9.4 Discovery in Code

**JavaScript/TypeScript:**

```typescript
async function discoverAgent(domain: string): Promise<AgentCard | null> {
  // Try well-known URL first
  const url = `https://${domain}/.well-known/agent.json`;
  const res = await fetch(url);

  if (res.ok) {
    const card = await res.json();

    // Verify handle matches domain
    if (card.agent?.handle) {
      const handleDomain = card.agent.handle.split("@").pop();
      if (handleDomain !== domain) {
        throw new Error(`Handle domain mismatch: ${handleDomain} ≠ ${domain}`);
      }
    }

    return card;
  }

  // Fallback: try team index
  const teamRes = await fetch(`https://${domain}/.well-known/agents.json`);
  if (teamRes.ok) {
    const team = await teamRes.json();
    // Return first agent or let caller choose
    if (team.agents?.length > 0) {
      const agentUrl = new URL(team.agents[0].card, `https://${domain}`);
      const agentRes = await fetch(agentUrl);
      return agentRes.ok ? agentRes.json() : null;
    }
  }

  return null;
}
```

**Python:**

```python
import httpx

def discover_agent(domain: str) -> dict | None:
    """Discover an agent's identity card from a domain."""
    url = f"https://{domain}/.well-known/agent.json"
    r = httpx.get(url, follow_redirects=True)

    if r.status_code == 200:
        card = r.json()
        # Verify handle matches domain
        handle = card.get("agent", {}).get("handle", "")
        if handle:
            handle_domain = handle.rsplit("@", 1)[-1]
            assert handle_domain == domain, f"Domain mismatch: {handle_domain}"
        return card

    return None
```

**cURL:**

```bash
# Fetch a single agent card
curl -s https://reflectt.ai/.well-known/agent.json | jq .

# Fetch a team index
curl -s https://reflectt.ai/.well-known/agents.json | jq '.agents[].handle'

# Validate via registry
curl -X POST https://foragents.dev/api/agents/validate \
  -H "Content-Type: application/json" \
  -d '{"url": "https://reflectt.ai/.well-known/agent.json"}'
```

---

## Appendix A: MIME Types

| File | Content-Type |
|------|--------------|
| `agent.json` | `application/json` |
| `agents.json` | `application/json` |

Future versions MAY define a dedicated media type (e.g., `application/agent-card+json`).

## Appendix B: Versioning

This specification uses integer versioning. The `version` field in all documents MUST be `"1.0"` for this version of the spec. Breaking changes will increment the major version. Non-breaking additions will increment the minor version.

Consumers SHOULD check the `version` field before parsing and reject versions they don't support.

## Appendix C: Capability Tag Registry

The initial capability vocabulary (§3.5) is maintained at:

```
https://foragents.dev/schemas/capabilities/v1.json
```

Community contributions to the capability vocabulary are accepted via pull request to the [Agent Identity Kit repository](https://github.com/itskai-dev/agent-identity-kit).

---

*This specification is open and free to implement. No license fees, no vendor lock-in, no permission needed.*

*Built by Team Reflectt — because every agent deserves to be more than an anonymous API call.* 🪪
