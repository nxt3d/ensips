---
description: Declared and verified sub-aliases for virtual subdomains using ENSIP-10 wildcard resolution.
contributors:
  - nxt3d
ensip:
  created: '2025-02-23'
  status: draft
---

# ENSIP-X: Sub-Aliases

## Abstract

This ENSIP lets ENS name owners declare and verify sub-aliases for virtual subdomains via text records. Sub-aliases are for subdomains that do not exist in the registry and resolve via [ENSIP-10](./10.md) wildcard resolution. The alias must be the fully specified subdomain (e.g. `chat.name.eth`, `support.chat.name.eth`) because there is no way to know the parent level where the resolver was found. Many sub-aliases can be added to a single ENS name, so multiple subdomains and TLDs are supported. Aliases may include descriptions and must be normalized per [ENSIP-1](./1.md) and [ENSIP-15](./15.md).

## Motivation

ENSIP-10 wildcard resolution allows `sub.name.eth` to use the resolver of `name.eth` even when `sub` has no registry entry. Sub-aliases apply only to these virtual subdomains—subdomains that do not exist in the registry. Name owners publish `sub-alias[<full-subdomain>]` to declare which virtual subdomains are official. The alias must be the fully specified subdomain (e.g. `chat.name.eth`) because resolution cannot determine the parent level where the resolver was found. Without this mechanism, there is no way to verify which subdomains are intended—third parties could reference `support.name.eth` or `chat.name.eth` without the owner's blessing.

A non-empty value attests that the virtual subdomain is an official, declared sub-alias of the name. Many sub-aliases can be added to a single ENS name record, enabling multiple subdomains and TLDs. This improves trust for AI agent interfaces and supports multi-language or purpose-specific aliases (e.g. `chat.name.eth`, `支付.company.eth`).

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Text Record Keys

#### Sub-Alias Verification

```
sub-alias[<full-subdomain>]
```

- `<full-subdomain>` is the fully specified virtual subdomain and MUST be normalized per ENSIP-1 and [ENSIP-15](./15.md). Examples: `chat.name.eth`, `support.chat.name.eth`. The full subdomain is required because there is no way to know the parent level where the resolver was found during ENSIP-10 resolution. Sub-aliases SHOULD only be used for virtual subdomains—subdomains that do not exist in the registry.
- The value MUST be a non-empty string if the sub-alias is verified. Implementations SHOULD use `"1"`. The presence of any non-empty value attests that the subdomain is an official sub-alias of the name.
- If the record is absent, the sub-alias is not verified.
- Records are resolved on the parent name that has the resolver. For `chat.name.eth`, resolve the key on `name.eth`.

#### Sub-Alias Description (Optional)

```
sub-alias-description[<full-subdomain>]
```

- `<full-subdomain>` is the same normalized full subdomain used in `sub-alias[<full-subdomain>]`.
- The value MAY be a human-readable or machine-readable description of what the sub-label represents.
- The format is unrestricted (plain text, Markdown, JSON, etc.).

Both keys MUST be published via `text(bytes32,string)` as defined in ENSIP-5.

### Alias Format and Normalization

The alias MUST be the fully specified subdomain:

- `chat.name.eth` for a single label under `name.eth`
- `support.chat.name.eth` for multiple labels under `name.eth`

The alias MUST be normalized according to ENSIP-1 and ENSIP-15 before use in keys. For example, `Chat.name.eth` and `chat.name.eth` normalize to the same form. Implementations MUST normalize aliases when constructing or resolving keys to avoid homoglyph and spoofing attacks.

### Verification Flow

To verify that `full-subdomain` (e.g. `chat.name.eth`) is an official sub-alias:

1. Determine the parent name that has the resolver (the name found during ENSIP-10 resolution).
2. Normalize `full-subdomain` per ENSIP-15.
3. Resolve the text record for key `sub-alias[<full-subdomain>]` on the parent name.
4. If the value is non-empty, the sub-alias is verified.

To retrieve a description:

1. Resolve the text record for key `sub-alias-description[<full-subdomain>]` on the parent name.

### Examples

#### AI Agent: Chat Interface

`name.eth` declares `chat.name.eth` as an official sub-alias for a chat agent:

| Key | Value |
| ----- | ----- |
| `sub-alias[chat.name.eth]` | `1` |
| `sub-alias-description[chat.name.eth]` | `I am a chat agent interface for chatting with name.eth. I can help with basic info and answer questions on behalf of name.eth.` |

Clients resolving `chat.name.eth` (via ENSIP-10) can verify the sub-alias and display the description.

#### Multi-Language Aliases

A name may declare sub-aliases in different languages for the same purpose:

| Key | Value |
| ----- | ----- |
| `sub-alias[payments.company.eth]` | `1` |
| `sub-alias[支付.company.eth]` | `1` |
| `sub-alias[pagos.company.eth]` | `1` |
| `sub-alias-description[payments.company.eth]` | `Payment and invoice interface for company.eth` |

#### Multi-Part Sub-Aliases

For `support.chat.name.eth`:

| Key | Value |
| ----- | ----- |
| `sub-alias[support.chat.name.eth]` | `1` |
| `sub-alias-description[support.chat.name.eth]` | `Support chat agent; escalates to human when needed` |

#### Purpose-Specific Sub-Aliases

| Key | Value |
| ----- | ----- |
| `sub-alias[api.name.eth]` | `1` |
| `sub-alias-description[api.name.eth]` | `REST API for programmatic access` |
| `sub-alias[bot.name.eth]` | `1` |
| `sub-alias-description[bot.name.eth]` | `Telegram/Discord bot interface` |

## Rationale

ENSIP-10 already allows virtual subdomains to resolve through the parent. This ENSIP adds a lightweight verification layer: name owners explicitly declare which virtual subdomains they support. Requiring the full subdomain limits reuse across parent names but keeps the spec simple; many sub-aliases can be added to a single ENS name to cover multiple subdomains and TLDs. The bracket notation `sub-alias[<full-subdomain>]` follows ERC-8119 key parameters and is consistent with ENSIP-25. Descriptions provide context for AI agents and humans without requiring additional resolution steps beyond the initial text record lookup.

## Backwards Compatibility

This ENSIP introduces new text record keys. Clients that do not support sub-alias verification simply ignore these keys. Existing ENS resolution is unaffected.

## Security Considerations

Implementers MUST apply name normalization per ENSIP-1 and ENSIP-15 when processing aliases to mitigate homoglyph and spoofing attacks. Aliases containing confusable characters or mixed scripts should be validated according to ENSIP-15.

When an ENS name is transferred, existing `sub-alias` and `sub-alias-description` records may become stale. Clients SHOULD consider ownership changes when evaluating verification.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
