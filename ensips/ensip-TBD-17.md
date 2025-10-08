---
title: Text Key Parameters
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-07-18
---

## Abstract

This ENSIP defines Text Key Parameters, an extension to ENS text record keys that allows for dynamic or contextual data to be stored and referenced by appending a parameter after a colon (`:`). This enables richer, more flexible key-value data for agentic systems, dApps, and other ENS-integrated applications.

## Motivation

While text record keys in ENS provide a namespace for data storage, many use cases require referencing data that is contextual, such as by timestamp, block number, user ID, or other parameters. Text Key Parameters allow for this flexibility, supporting composable and dynamic data models for both global keys and service keys.

## Specification

Text record keys in ENS can be global keys (strings without dot notation) or follow the reverse dot notation format (e.g., `com.twitter`, `org.telegram`). Text Key Parameters extend these keys by allowing a parameter to be appended after a colon (`:`).

### Syntax

```
{text-key}:{parameter}
```

Where:
- `{text-key}` is a text record key as defined in ENSIP-5
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

Text Key Parameters enable precise, context-aware data retrieval in ENS. For example, a text record can request the latest or historical ETH price from an oracle by specifying a block number parameter, or a profile can have different avatars for different years:

This approach allows clients to deterministically query external data sources (such as oracles) for time-specific or event-specific data, supporting advanced analytics, historical queries, and agentic systems that require verifiable context. Text Key Parameters make ENS a more powerful and flexible data registry for both human and machine consumers.

## Backwards Compatibility

Unaware clients will simply ignore the parameter and treat the full key as a string. Existing behavior is unaffected.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
