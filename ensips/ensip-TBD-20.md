---
title: AI Agent Registry ENS Name Verification
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-10-02
---

# ENSIP-TBD-20: AI Agent Registry ENS Name Verification

## Abstract

This ENSIP defines a standardized method for verifying ENS names associated with AI Agent registries using text records with the format "agent-registry:<ERC-7930 Chain ID>". The specification enables bidirectional verification between an AI agent registry and its corresponding ENS name, ensuring consistency and trust in the AI agent ecosystem. This format enables multichain agents, allowing a single ENS name to reference agent registries across multiple chains.

## Motivation

As AI agents become more prevalent in decentralized systems, there is a growing need for standardized verification methods that link AI agent registries to ENS names. This verification process is essential for:

- Establishing trust between an AI agent registry and its claimed ENS identity
- Enabling cross-chain AI agent registry discovery and verification
- Providing a standardized method for AI agent registry implementations
- Supporting both registry-to-ENS and ENS-to-registry verification flows

The current lack of standardized verification methods creates fragmentation and potential security issues in the AI agent ecosystem. This ENSIP addresses these concerns by providing a clear specification for AI agent registry verification using ENS text records. By including the chain specification in the record key using ERC-7930 format, a single ENS name can reference multiple agent registries across different chains, enabling multichain agent implementations.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Agent Registry Text Record

This ENSIP introduces text records with the format `agent-registry:<ERC-7930 Chain ID>` that store verification data for an AI agent registry on a specific chain. The record key MUST use the format `agent-registry:` followed by an ERC-7930 Chain ID, which is an ERC-7930 Interoperable Address V1 binary format with address length 0 (no address), representing only the chain specification. The ERC-7930 Chain ID in the record key MUST be hex-encoded without the `0x` prefix.

The record value MUST contain a hex-encoded string representing the ERC-7930 Interoperable Address V1 binary format (with the full address) appended with a one-byte agentID length followed by the hex-encoded agentID bytes.

The format is a hex-encoded string containing: `ERC-7930-Binary-Address + agentIDLength (1 byte) + agentID (variable length)`

### Agent Registry Record Format

The agent-registry text record contains the ERC-7930 Interoperable Address V1 format followed by the agentID:

```
┌─────────────────────┬───────────────┬───────────┐
│ ERC-7930 address    │ AgentIDLength │ AgentID   │
└─────────────────────┴───────────────┴───────────┘
```

### ERC-7930 Chain ID Format

The ERC-7930 Chain ID in the record key is an ERC-7930 Interoperable Address V1 binary format with address length 0 (no address), representing only the chain specification. See [ERC-7930 Example 4](https://eips.ethereum.org/EIPS/eip-7930) for the format specification.

### ERC-7930 Interoperable Address V1 Format

The ERC-7930 Interoperable Address V1 binary format in the record value is used to represent the full cross-chain address (including the registry address). See [ERC-7930](https://eips.ethereum.org/EIPS/eip-7930) for the complete format specification.

### Verification Methods

This ENSIP defines two verification flows:

#### Registry-to-ENS Verification

When starting from an AI agent registry:

1. The client MUST query the ENS name of the agent using the agent metadata
2. The client MUST construct the ERC-7930 Chain ID format (with address length 0) for the chain
3. The client MUST forward resolve the ENS text record `agent-registry:<ERC-7930 Chain ID>` for the specified ENS name
4. The client MUST parse the hex-encoded data to extract the ERC-7930 address and agentID
5. The client MUST verify that the ERC-7930 address corresponds to the expected registry
6. The client MUST verify that the agentID corresponds to the expected agent in the registry

#### ENS-to-Registry Verification

When starting from an ENS name:

1. The client MUST determine the target chain ID for the agent registry
2. The client MUST construct the ERC-7930 Chain ID format (with address length 0) for the target chain
3. The client MUST query the `agent-registry:<ERC-7930 Chain ID>` text record for the ENS name
4. The client MUST parse the hex-encoded data to extract the ERC-7930 address and agentID
5. The client MUST verify that the registry contract is the expected AI agent registry
6. The client MUST verify that the registry's claimed ENS name matches the original ENS name

### Registry Standards

Registries implementing this ENSIP SHOULD follow established standards such as ERC-8004 for AI agent registry implementations. The specific registry standard used MUST be documented and accessible to clients for proper verification.

## Examples

### Example Agent Registry Record

For an AI agent registry on Ethereum Mainnet (EIP-155 chain ID 1) with:
- ERC-7930 Chain ID (for key): `0x000100000100` (Ethereum Mainnet with no address, see [ERC-7930 Example 4](https://eips.ethereum.org/EIPS/eip-7930))
- ERC-7930 Address (for value): `0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8` (full address, see [ERC-7930 Example 1](https://eips.ethereum.org/EIPS/eip-7930))
- Agent ID: `42` (variable length, 1 byte = 0x2A)

The text record key would be: `agent-registry:000100000100`

The text record value would contain:
```
0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA82a
```

### Example Multichain Agent

An ENS name can reference multiple agent registries across different chains using [ERC-7930](https://eips.ethereum.org/EIPS/eip-7930) format:

- `agent-registry:000100000100` → Ethereum Mainnet (EIP-155 chain ID 1) registry
- `agent-registry:00010000018900` → Polygon (EIP-155 chain ID 137) registry
- `agent-registry:00010002A86A00` → Avalanche (EIP-155 chain ID 43114) registry

Each record contains the ERC-7930 address of the registry on that specific chain along with the agentID.

## Rationale

This ENSIP provides a standardized method for AI agent registry verification using the established ERC-7930 format for cross-chain addresses. The design prioritizes simplicity and interoperability by using text records with variable-length agentID encoding, enabling bidirectional verification between registries and ENS names while maintaining compatibility with existing ENS infrastructure. By including the chain specification in the record key using ERC-7930 Chain ID format (with address length 0), a single ENS name can reference multiple agent registries across different chains, enabling multichain agent implementations. This allows agents to operate across multiple blockchain networks while maintaining a single ENS identity.

## Backwards Compatibility

This ENSIP introduces a new text record type and does not affect existing ENS functionality. It is fully backward compatible with all existing ENS records and resolvers. Existing ENS names without "agent-registry" records will continue to function normally.

## Security Considerations

None.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
