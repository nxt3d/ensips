---
title: ENS Text Record URI Scheme
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-07-18
---

## Abstract

This ENSIP introduces the `enstr` URI scheme for referencing ENS text records. The scheme provides a standardized way to address and embed ENS text record data, enabling modular, interoperable context composition for agentic systems and other ENS-integrated applications.

## Motivation

ENS text records are key-value pairs associated with ENS names, but there is currently no standard URI format for referencing them. The `enstr` URI scheme enables direct, verifiable linking to specific text records, supporting richer context and cross-ENS data composition for wallets, agents, dApps, and middleware.

## Specification

### Syntax

The `enstr` URI scheme follows this format:

```
enstr:{ens-name}:{text-record-key}
```

Where:
- `{ens-name}` is a valid ENS name (e.g., `ens.eth`, `example.eth`)
- `{text-record-key}` is the key of the text record to resolve

### Examples

```
enstr:ens.eth:eth.ens.dao-mission
enstr:vitalik.eth:com.twitter
enstr:example.eth:description
enstr:token.eth:token-address:pepe
```

### Usage Formats

#### Raw URI Format

Text records can be referenced directly using the `enstr` URI:

```
enstr:price-oracle.eth:eth-price
```

#### JSON Format with Metadata

Text records can be wrapped in JSON for additional context:

```json
{
  "text-record": "enstr:price-oracle.eth:eth-price",
  "description": "The current price of Ethereum in USD from the Chainlink Oracle on L1 Ethereum, with 6 decimals of precision and no decimal characters (1000000 = $1.00)."
}
```

#### Markdown Link Format

Text records can be embedded as Markdown links:

```markdown
[Current ETH price from Chainlink Oracle with 6 decimals precision](enstr:price-oracle.eth:eth-price)
```

#### Service Key Parameters

For contextual and dynamic references, Service Key Parameters (see [ENSIP-TBD-17](./ensip-TBD-17.md)) can be used to append parameters such as timestamps, block numbers, or other identifiers to the text record key:

```
enstr:data.eth:com.chainlink.ether.price:20250718      # Request ETH price for July 18, 2025
enstr:data.eth:com.chainlink.ether.price:block:20000000 # Request ETH price for block 20,000,000
```

## Rationale

A standardized URI scheme for ENS text records enables composable, machine-readable context for agentic systems, dApps, and other ENS-integrated tools. It supports modularity, discoverability, and interoperability across the ENS ecosystem.

## Backwards Compatibility

Unaware clients will simply ignore the new URI scheme. Existing behavior is unaffected.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
