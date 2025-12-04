---
description: A standard text record for resolver metadata including version, upgrade status, and update information
contributors:
  - premm.eth
ensip:
  created: '2025-12-04'
  status: draft
---

# Resolver Info Text Record

## Abstract

This ENSIP extends [ENSIP-5: Text Records](https://docs.ens.domains/ensip/5) by defining a standard text record key that allows resolvers to expose metadata about themselves. This enables clients, users, and AI agents to discover resolver version information, upgrade status, planned updates, authors, and change history. AI agents primarily use this information to verify resolver claims and assess security, facilitating better resolver management and client decision-making.

## Motivation

As the ENS ecosystem grows, various resolver implementations have emerged with different capabilities, versions, and upgrade paths. Currently, there is no standardized way for clients to discover key information about resolvers such as their version, upgradeability, maintenance schedules, or development history. This lack of standardization makes it difficult for users and AI agents to make informed decisions about resolver reliability and security.

This ENSIP defines a standard text record key that resolvers can use to expose metadata about themselves. This enables better decision-making about resolver reliability, easier debugging, and allows AI agents to verify resolver claims and assess security.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Text Record Key

This ENSIP defines a new global key for use with the `text()` function specified in [ENSIP-5](https://docs.ens.domains/ensip/5):

**`resolver-info`** - Metadata about the resolver implementation

### Format

The `resolver-info` text record format is unstructured and flexible. Resolvers MAY format the content in any way they choose, such as Markdown, JSON, plain text, or any combination thereof. The following examples demonstrate common formatting approaches.

The `resolver-info` MAY contain the following information:

- **Version**: The version identifier of the resolver implementation. This SHOULD follow semantic versioning (e.g., "1.2.3") but MAY use any versioning scheme the resolver implementer chooses.

- **Upgradable**: Information about whether the resolver contract is upgradable and what upgrade mechanism it uses (e.g., UUPS, Transparent Proxy, immutable, etc.).

- **Authors**: Information about the authors or maintainers of the resolver. This MAY include names, ENS names, GitHub handles, or other identifiers.

- **Update Info**: Information about planned updates, maintenance windows, or deprecation notices. This MAY include preview implementation addresses (optionally as ERC-7930 Interoperable Address format, which can be on a testnet) and a bytecode hash (keccak256 of the contract bytecode). Clients can verify the bytecode hash to whitelist upgrades before they occur.

- **Change History**: A history of changes, updates, or version history for the resolver. This MAY be presented as a list, table, or structured format.

- **Features**: A list of features or capabilities supported by this resolver, presented as a Markdown section (e.g., multichain, CCIP-Read, wildcard).

### Example

The following example demonstrates a possible formatting approach using Markdown with embedded JSON:

````markdown
# Resolver Information

**Version:** 2.1.0  
**Upgradable:** Yes

## Authors
- premm.eth

## Change History
- **v2.1.0** (2025-01-15): Added CCIP-Read support - `0x1234567890123456789012345678901234567890`
- **v2.0.0** (2024-12-01): Initial release with multichain support - `0xabcdefabcdefabcdefabcdefabcdefabcdefabcd`

## Structured Data

```json
{
  "version": "2.1.0",
  "upgradable": "UUPS",
  "authors": ["premm.eth"],
  "features": ["multichain", "ccip-read", "wildcard"],
  "updateInfo": {
    "date": "2025-02-15",
    "previewImplementation": "0x00010000010114987654321098765432109876543210",
    "bytecodeHash": "0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890"
  }
}
```
````

### Resolver Behavior

Resolvers implementing this ENSIP:

- MAY format the `resolver-info` content in any way they choose (Markdown, JSON, plain text, etc.)
- MAY include any of the suggested sections (Version, Upgradable, Authors, Update Info, Change History, Features) or any other information they deem relevant
- SHOULD update the `resolver-info` record when the resolver is upgraded or when significant changes occur
- MAY return an empty string if the resolver does not wish to expose this information (clients MUST handle empty strings gracefully)

### Client Behavior

Clients, users, and AI agents querying the `resolver-info` record:

- SHOULD get the key data about the resolver to assist in reviewing the resolver for security. This data SHOULD NOT override ERC-165 or [ENSIP-22: Contract Features](https://docs.ens.domains/ensip/22). The key idea of this field is that it is human-readable and provides a place for metadata about the contract such as scheduled upgrades
- MUST handle cases where the field is not set (empty string returned), malformed, or does not conform to this specification
- MAY parse JSON code blocks if present for programmatic access

## Rationale

### Why a Text Record?

Using the existing text record mechanism from ENSIP-5 provides several advantages:
- No new resolver interface required - works with all existing resolvers that support ENSIP-5
- Leverages existing infrastructure and tooling

## Backwards Compatibility

This proposal extends ENSIP-5 by defining a new global key with no breaking changes.

## Security Considerations

Clients should be aware that `resolver-info` data is provided by the resolver itself, may not always be accurate.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

