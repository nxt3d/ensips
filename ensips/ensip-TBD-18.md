---
title: Chain-ID Text Record
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-08-07
---

## Abstract

This ENSIP, which extends [ENSIP-5: Text Records](https://docs.ens.domains/ensip/5), defines a new global text record, `chain-id`, for ENS resolvers. The `chain-id` record provides a standardized way to store and resolve chain IDs for crosschain addresses, enhancing interoperability across different blockchains.

## Motivation

As blockchain ecosystems grow, the need for crosschain interactions becomes pressing. A standardized method to resolve chain IDs is necessary for applications that require crosschain address resolution. This ENSIP proposes the `chain-id` text record, allowing for crosschain address resolution.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Text Record Key

* **Key**: `chain-id`
* **Value**: An EVM chain ID
* **Format**: The chain ID MUST be stored as a UTF-8 text string in hex format, excluding the 0x prefix.

This record allows for the resolution of crosschain addresses, such as `vitalik.eth@avid.cid.eth`, or in the case where `cid.eth` is assumed, `vitalik.eth@avid`. This standard precedes a standard for crosschain addresses; however, it predicts that ENS can be used to resolve a chain ID.

## Backwards Compatibility

Unaware clients will simply ignore the new key; existing behavior is unaffected.

## Security Considerations

There are no security considerations specific to this ENSIP.

## Rationale

This ENSIP creates a standardized text record for resolving chain IDs associated with ENS names. New standards for crosschain addresses are expected to be defined using ENS, and it is necessary to have a standardized way to resolve chain IDs. This ENSIP lays the groundwork for crosschain address standards to be defined using ENS.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
