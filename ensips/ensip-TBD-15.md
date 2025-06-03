---
title: Agent Delegations
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-06-03
---

## Abstract

This ENSIP defines two complementary **patterned text records** that establish a parent ↔ delegate relationship between **ENS names** (not addresses):

* **`agent-delegate:<ENS name>`** — asserted on the *parent* (owner) ENS name to nominate one or more delegate agents.
* **`agent-parent:<ENS name>`** — asserted on the *delegate* ENS name to acknowledge a specific parent.

When both records are present with non‑empty values, the pair of names is considered an **active delegation**. The record values are unspecified UTF‑8 but can be written for LLM consumption (e.g., using Markdown with embedded JSON or YAML), though the format is optional so that downstream agent frameworks can read guard‑rails, scopes, or policies.

The design is inspired by the IETF draft *OAuth AI Agents On Behalf of User* (draft‑oauth‑ai‑agents‑on‑behalf‑of‑user‑00) and brings a similar framework to ENS‑resolved identities.

## Motivation

As AI agents become first‑class actors on blockchains, they need a **verifiable, on‑chain mechanism** to declare *who* they may act for and *under what limits*. Existing ENS text records allow arbitrary key/value storage but lack structured conventions for delegation.

By introducing paired keys that must be reciprocally set, this ENSIP enables:

* Wallets and dApps to verify that a delegate name truly operates on behalf of a parent.
* Owners to publish fine‑grained scopes (e.g. read‑only, spend cap, chain allow‑list) in human‑ and machine‑readable form.
* Multi‑parent / multi‑delegate graphs (many‑to‑many) without additional smart‑contract complexity.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### 1. Text‑record key patterns

| Role     | Key pattern                 | Example (parent = `alice.eth`, delegate = `bob.bot.eth`) |
| -------- | --------------------------- | -------------------------------------------------------- |
| Parent   | `agent-delegate:<delegate>` | `agent-delegate:bob.bot.eth`                             |
| Delegate | `agent-parent:<parent>`     | `agent-parent:alice.eth`                                 |

* Keys **MUST** be published via `text(bytes32,string)` (ENSIP‑5).
* The substring after the colon **MUST** be the ENS name in its normalized (lowercase) form.
* Record **values** **MUST NOT** be empty when the delegation is active. Setting an empty string on either side **SHALL** terminate the delegation.

### 2. Recommended value format

Values are unspecified UTF‑8. Markdown may be used so LLMs can ingest the content easily, but it is not required.

Structured data (rules, caps, allow‑lists) **may** be wrapped in standard fenced code blocks (e.g., ` ```json` or ` ```yaml`) for readability.

```markdown
You are a read‑only analytics agent operating under strict usage constraints.

<code>
{
  "maxTxPerDay": 3,
  "spendCapUSD": 0,
  "allowedChains": ["ethereum", "optimism"],
  "expires": "2026-05-30T00:00:00Z"
}
</code>
```

### 3. Delegation lifecycle

1. **Establish** — Parent publishes `agent-delegate:<delegate>` with non‑empty value **AND** delegate publishes `agent-parent:<parent>` with non‑empty value.
2. **Verify** — Clients resolve both names and confirm reciprocal keys & non‑empty values.
3. **Terminate** — Either party sets its corresponding key to the empty string `""`.

### 4. Examples

#### Parent side (`alice.eth`)

*(See Section 2 for example)*

#### Delegate side (`bob.bot.eth`)

```markdown
Acting on behalf of `alice.eth` to monitor on‑chain governance and publish a daily summary.

<code>
{
  "actingFor": "alice.eth",
  "intent": "governance-summary",
  "expires": "2025-12-31T00:00:00Z"
}
</code>
```

### 5. Optional root declaration keys&#x20;

The following **root‑level** text records (without an appended ENS name) are provided for informational listings. Their values are unspecified UTF‑8 and may be written for easy consumption by humans and LLMs, but **they do not create binding delegations**. Only the reciprocal patterned keys (`agent‑delegate:<name>`, `agent‑parent:<name>`) establish authority under this ENSIP.

| Key              | Published by | Notes                                                          |
| ---------------- | ------------ | -------------------------------------------------------------- |
| `agent-delegate` | Parent       | Optional prose or structured data listing all delegate agents. |
| `agent-parent`   | Delegate     | Optional prose or structured data listing all parent names.    |

Example parent listing:

```markdown
# Agent Delegation Overview

I have delegated to `alpha.bot.eth`, `beta.bot.eth`, and `gamma.bot.eth`.  
These agents may execute on‑chain governance research and publish summary reports.
```

Example delegate listing:

```markdown
# Acting On Behalf Of

Currently serving `alice.eth`, `bravo.eth`, and `charlie.eth`.  
Primary role: event planner.
```

## Rationale

* **Name‑to‑name linkage** keeps delegation *human‑readable* and re‑useable across chains.
* Patterned keys (`agent-delegate:<delegate>`, `agent-parent:<parent>`) avoid exploding the ENS key‑space while remaining queryable.
* Requiring reciprocal, non‑empty records prevents unilateral impersonation.
* Leveraging Markdown + JSON enables immediate LLM usage without additional on‑chain schema work.
* **Composability**: Any parent or delegate may itself publish further `agent-delegate:<delegate>` or `agent-parent:<parent>` keys, forming an arbitrarily deep “agent web.” This mirrors real‑world delegation chains (e.g., a wedding planner can plan the wedding of her lawyer) and allows complex trust hierarchies without extra smart‑contract layers.

## Backwards Compatibility

Unaware clients treat unknown keys as opaque text (per ENSIP‑5). Existing behavior is unchanged.

## Security Considerations

There are no additional security considerations introduced by this ENSIP. 

## Copyright

Copyright and related rights waived [vi](https://creativecommons.org/publicdomain/zero/1.0/)[a ](https://creativecommons.org/publicdomain/zero/1.0/)[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
