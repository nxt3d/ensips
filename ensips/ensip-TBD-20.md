---
title: AI Agent Registry ENS Name Verification
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-10-02
---

# ENSIP-TBD-20: AI Agent Registry ENS Name Verification

## Abstract

This ENSIP defines a standardized method for verifying ENS names associated with AI Agent registries using a new text record type called "agent-registry". The specification enables bidirectional verification between an AI agent registry and its corresponding ENS name, ensuring consistency and trust in the AI agent ecosystem.

## Motivation

As AI agents become more prevalent in decentralized systems, there is a growing need for standardized verification methods that link AI agent registries to ENS names. This verification process is essential for:

- Establishing trust between an AI agent registry and its claimed ENS identity
- Enabling cross-chain AI agent registry discovery and verification
- Providing a standardized method for AI agent registry implementations
- Supporting both registry-to-ENS and ENS-to-registry verification flows

The current lack of standardized verification methods creates fragmentation and potential security issues in the AI agent ecosystem. This ENSIP addresses these concerns by providing a clear specification for AI agent registry verification using ENS text records, where each ENS name corresponds to exactly one registry.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Agent Registry Text Record

This ENSIP introduces a new text record type called "agent-registry" that stores verification data for an AI agent registry. The record MUST contain a hex-encoded string representing the ERC-7930 Interoperable Address V1 binary format appended with a one-byte agentID length followed by the hex-encoded agentID bytes.

The format is a hex-encoded string containing: `ERC-7930-Binary-Address + agentIDLength (1 byte) + agentID (variable length)`

### Agent Registry Record Format

The agent-registry text record contains the ERC-7930 Interoperable Address V1 format followed by the agentID:

```
┌─────────────────────┬───────────────┬───────────┐
│ ERC-7930 address    │ AgentIDLength │ AgentID   │
└─────────────────────┴───────────────┴───────────┘
```

### ERC-7930 Interoperable Address V1 Format

The ERC-7930 Interoperable Address V1 binary format is used to represent cross-chain addresses in a standardized way. This format allows for consistent representation of addresses across different blockchain networks and provides the necessary information for cross-chain verification.

### Verification Methods

This ENSIP defines two verification flows:

#### Registry-to-ENS Verification

When starting from an AI agent registry:

1. The registry defines an ENS name and MAY specify a version number
2. The client MUST forward resolve the ENS text record "agent-registry" for the specified ENS name
3. The client MUST parse the hex-encoded data to extract the ERC-7930 address and agentID
4. The client MUST verify that the ERC-7930 address corresponds to the expected registry
5. The client MUST verify that the agentID corresponds to the expected agent in the registry

#### ENS-to-Registry Verification

When starting from an ENS name:

1. The client MUST query the "agent-registry" text record for the ENS name
2. The client MUST parse the hex-encoded data to extract the ERC-7930 address and agentID
3. The client MUST verify that the registry contract is the expected AI agent registry
4. The client MUST verify that the registry's claimed ENS name matches the original ENS name
5. The client MUST verify that the version is 1 or no version number is specified

### Registry Standards

Registries implementing this ENSIP SHOULD follow established standards such as ERC-8004 for AI agent registry implementations. The specific registry standard used MUST be documented and accessible to clients for proper verification.

### Version Specification

This ENSIP defines version 1 (v1) of the AI agent registry verification standard. To follow this specification, the version MUST be 1 or no version number may be specified. Future versions may introduce additional fields or modify the verification process. Version information SHOULD be included in registry implementations to ensure proper client compatibility.

## Examples

### Example Agent Registry Record

For an AI agent registry with:
- ERC-7930 Address: `0x1234567890abcdef...` (hex-encoded binary format)
- Agent ID: `42` (variable length, 1 byte in this case)

The "agent-registry" text record would contain:
```
0x1234567890abcdef...012a
```

## Rationale

This ENSIP provides a standardized method for AI agent registry verification using the established ERC-7930 format for cross-chain addresses. The design prioritizes simplicity and interoperability by using a single text record with variable-length agentID encoding, enabling bidirectional verification between registries and ENS names while maintaining compatibility with existing ENS infrastructure.

## Backwards Compatibility

This ENSIP introduces a new text record type and does not affect existing ENS functionality. It is fully backward compatible with all existing ENS records and resolvers. Existing ENS names without "agent-registry" records will continue to function normally.

## Security Considerations

None.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
