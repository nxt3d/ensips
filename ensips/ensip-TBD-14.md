---
title: Agentic Systems Interface
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-05-17
---

## Abstract

ENSIPXX introduces the `root-context` text record, which provides a universal entry point to ENS names for agentic systems. This ENSIP defines a standard way to specify interfaces using the `root-context` text record, describing *what* the name represents (data source, chatbot, autonomous agent, etc.) and *how* to interact with it. By storing context data onchain via ENS, any app (chat front ends, wallet UIs, MCP middleware, crawlers) has a reliable, verifiable place to discover AI context data. Using ERC 3668 it is also possible to store the `root-context` text record offchain. However, any offchain values must be fully verified onchain to comply with this ENSIP.

## Motivation

Agentic systems that require verifiable context data are emerging, including agents that can propose blockchain transactions with calldata, or that rely on critical context data requiring verifiability. ENS names are well positioned to register verifiable context data because they can be stored onchain and are supported by existing tooling. ENSIPXX introduced the `root-context` text record as a starting point for agentic context data but did not define any format standards. This ENSIP introduces a standard way to discover interfaces and interface implementations for storing verifiable AI context. Using this standard, agentic systems can be developed using any type of interface, while allowing apps to ignore interfaces they do not require.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Agentic System Interfaces

This section defines an interface standard for the `root-context` text record of ENS names, allowing for the definition of multiple interfaces for agentic systems.

This ENSIP allows for the `root-context` text to gain the necessary context to interact with the ENS name from different types of clients, as well as direct clients to find further context data.

## Links to Text Records

It is possible to link to other text records from the `root-context` text record, allowing for data to be updated independently from the main `root-context` manifest. Other text records of the same ENS name MUST be specified using `text-record-key` and `text-record-description` pairs. The `text-record-key` is a unique identifier key for the text record, and the `text-record-description` is a human-readable description of the text record, which can be used to provide context for the data of the text record and instruct clients how to use the data.

```
text-record-key: eth-price
text-record-description: The current price of Ethereum in USD from the Chainlink Oracle on L1 Ethereum, with 6 decimals of precision and no decimals characters (1000000 = $1.00).
```

## Head Matter

The first part of the `root-context` must be head matter, which any client MUST read first. It may contain metadata, an index of interfaces, or other context, which can be interpreted by agentic systems semantically, and can also provide context for which interfaces are available and how they can be used.

## Interfaces

Next, implementers MAY embed one or more interfaces in their `root-context` manifest using the `----- Interface Name -----` delimiter format, including an optional space and five dashes on each side of the interface name. An interface name MUST NOT appear more than once.

```
interfaces: [chat, agent, tools]
summary: This ENS name provides a comprehensive ExampleProject support system with chat capabilities, agent functionality for API interactions, and specific tools for status queries and report generation.

interface_summaries:
  chat: Official support bot for ExampleProject with warm greetings, giving friendly instructions.
  agent: MCP middleware agent for API queries, user management, and authenticated actions
  tools: Status queries and report generation with timestamp parameters

--- Chat ---

You are the official support bot for ExampleProject. Greet users warmly and answer questions about our API, pricing, and getting started. Use the documentation linked below for accurate information.
. . .

--- Agent ---

You are a MCP middleware agent for ExampleProject. Users may ask you to swap tokens for Eth, the native token of the Ethereum blockchain. The first step in completing a swap is for you to determine which token the user wants to swap with Eth. You can do this by using a standard API connection to a coin listing service like CoinGecko.

To verify the API response, use the official name of the token and the symbol of the token, in the format `token-address:token-name:token-symbol` with no spaces. Resolve the text record for the token address and verify the token address matches the token name and symbol. For example,

```

text-record-key: token-address\:pepe\:pepe
text-record-description: The token address for the PEPE token on the Ethereum blockchain. There are many PEPE tokens, but this is the one deployed on the Ethereum blockchain on April 14, 2023, and reached a market cap of over \$8 billion.

```
. . .

--- Tools ---

-- status --

Queries the ExampleProject API with the given parameters and returns structured data.

Parameters: none

-- generate-report --

Generates a usage report for a given time period.

Parameters:
- start: Start time as Unix timestamp
- end: End time as Unix timestamp
```

## Client Resolution Flow for Interfaces

1. Resolve the `root-context` text record for the target ENS name.
2. Read the head matter, interpret the metadata, instructions, and context data.
3. If the head matter instructs the client to read one or more interfaces, parse the manifest to identify the interfaces using the `----- Interface Name -----` delimiters and go to step 4. If not, end on step 3.
4. Read the final context data from one or more interfaces.

## ENS Text Record URI Scheme

This ENSIP introduces a new URI scheme for referencing ENS text records: `enstr`. The scheme allows for direct addressing of specific text records on ENS names, providing a standardized way to link to and embed ENS text record data within interfaces.

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
enstr:token.eth:token-address:pepe:pepe
```

### Resolution Requirements

When an `enstr` URI is encountered within an interface, clients MUST:

1. Parse the URI to extract the ENS name and text record key
2. Resolve the specified text record for the given ENS name
3. Include the resolved text record value in the context provided to the agentic system
4. Handle resolution failures gracefully by noting the URI as unresolvable

### Usage in Interfaces

The `enstr` URI scheme can be embedded within interface specifications to reference external text record data. This allows for modular context composition where different pieces of data can be maintained independently across different ENS names.

Example usage within an interface:

```
--- Agent ---

You are an agent for token swapping. When verifying token information, always check the official token data at enstr:token.eth:token-address:pepe:pepe for the canonical PEPE token address.

Additional context available at:
- Token description: enstr:token.eth:description  
- Token website: enstr:token.eth:url
```

## Links

Decentralized storage protocols use content identifiers (CIDs) instead of URLs, such that the link is also a hash of the content. Interfaces MAY include embedded links and content in the form of CIDs, DataURLs, `enstr` URIs, or other types of links. DataURLs and `enstr` URIs MUST be resolved and included in the context received by the client. DataURLs allow for images to be encoded and included in a raw text document. Other types of deterministic URIs, such as IPFS CIDs, MAY be queried and included in the context received by clients. Standard URLs, which cannot be verified, SHOULD NOT be queried or included in the context. However, all URIs including URLs MUST be included in the context as a reference (e.g., as a literal URL string).

## Markdown

Markdown provides useful formatting features that can be used within an interface specification. For example, it can be useful to embed JSON or code examples within an interface using a fenced code block:

```json
{
  "name": "Swap Agent",
  "date": "05-30-2025"
}
```

Headers and text styling such as bold, italics, etc., can also be used.

## Rationale

This ENSIP builds upon ENSIPXX, which introduced the `root-context` text record, and defines a standardized interface format for AI systems. The interface-based approach allows a single ENS name to support multiple AI use cases while maintaining a clean separation of concerns.

By specifying a structured format for `root-context`, this ENSIP enables developers to create more predictable and interoperable agentic systems, reducing the need for ad hoc parsing or proprietary formats. It builds a foundation for ecosystem-wide conventions that can benefit wallet developers, front end builders, middleware, and even cross platform agent orchestration. This ensures ENS names can act not just as identity anchors, but as context rich agents.

## Backwards Compatibility

Unaware clients will simply ignore the new key. Existing behavior is unaffected.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
