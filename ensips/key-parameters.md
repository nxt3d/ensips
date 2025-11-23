---
title: Key Parameters
description: An extension to ENS record keys (text and data) that allows for dynamic or contextual data to be stored and referenced by appending a parameter after a colon
contributors: 
    - premm.eth
ensip:
  created: "2025-07-18"
  status: draft
---

# ENSIP-XX: Key Parameters

## Abstract

This ENSIP defines Key Parameters, an extension to ENS record keys (both text records via ENSIP-5 and data records via ENSIP-24) that allows for dynamic or contextual data to be stored and referenced by appending a parameter after a colon (`:`). This enables richer, more flexible key-value data for agentic systems, dApps, and other ENS-integrated applications.

## Motivation

While record keys in ENS (both text records via ENSIP-5 and data records via ENSIP-24) provide a namespace for data storage, many use cases require referencing data that is contextual, such as by timestamp, block number, user ID, or other parameters. Key Parameters allow for this flexibility, supporting composable and dynamic data models for both global keys and service keys.

## Specification

Record keys in ENS can be global keys (strings without dot notation) or follow the reverse dot notation format (e.g., `com.twitter`, `org.telegram`). Key Parameters extend these keys by allowing a parameter to be appended after a colon (`:`), and apply to both text records (ENSIP-5) and data records (ENSIP-24).

### Syntax

```
{key}:{parameter}
```

Where:
- `{key}` is a record key as defined in ENSIP-5 (for text records) or ENSIP-24 (for data records)
- `{parameter}` is a contextual value, such as a timestamp, block number, user ID, etc.

### Examples

```
com.chainlink.ether.price:20250718      # Service key with date parameter
com.chainlink.ether.price:block:20000000 # Service key with block parameter
com.example.users:alice               # Service key with user parameter
com.example.groups:public:2025        # Service key with year parameter
avatar:2025                           # Global key with year parameter
bio:en                                # Global key with language parameter
```

Parameters are arbitrary UTF-8 string, and may include any number of special characters, for example full URLs are possible. 

```
org.neo.uri:https://neo.org/api/v1/resource?foo=bar&baz=qux  # Service key with URL parameter
website:https://example.com                                # Global key with URL parameter
```

## Rationale

Key Parameters enable precise, context-aware data retrieval in ENS. For example, a record can request the latest or historical ETH price from an oracle by specifying a block number parameter, or a profile can have different avatars for different years.

This approach allows clients to deterministically query external data sources (such as oracles) for time-specific or event-specific data, supporting advanced analytics, historical queries, and agentic systems that require verifiable context. Key Parameters make ENS a more powerful and flexible data registry for both human and machine consumers.

## Backwards Compatibility

Existing behavior is unaffected.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
