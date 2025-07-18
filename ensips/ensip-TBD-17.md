---
title: Service Key Parameters
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-07-18
---

## Abstract

This ENSIP defines Service Key Parameters, an extension to ENS text record Service Keys that allows for dynamic or contextual data to be stored and referenced by appending a parameter after a colon (`:`). This enables richer, more flexible key-value data for agentic systems, dApps, and other ENS-integrated applications.

## Motivation

While Service Keys in ENS text records provide a namespace for service-specific data, many use cases require referencing data that is contextual, such as by timestamp, block number, user ID, or other parameters. Service Key Parameters allow for this flexibility, supporting composable and dynamic data models.

## Specification

Service Keys are defined in ENSIP-5 as keys in reverse dot notation for a namespace which the service owns (e.g., `com.twitter`, `org.telegram`). Service Key Parameters extend this by allowing a parameter to be appended after a colon (`:`).

### Syntax

```
{service-key}:{parameter}
```

Where:
- `{service-key}` is a valid Service Key as defined in ENSIP-5
- `{parameter}` is a contextual value, such as a timestamp, block number, user ID, etc.

### Examples

Using ensip-TBD-16, it is possible to display text records as fully deterministic URIs using the `enstr` URI scheme. 

```
enstr:data.eth:com.chainlink.eth.price:20250718      # Price at a specific date
enstr:data.eth:com.chainlink.eth.price:block:20000000 # Price at a specific block
enstr:api.eth:com.example.users:alice               # Data for user 'alice'
enstr:api.eth:com.example.groups:public:2025        # Public group data for year 2025
```

Parameters are arbitrary UTF-8 string, and may include any number of special characters, for example full URLs are possible. 

```
enstr:file.eth:org.neo.uri:https://neo.org/api/v1/resource?foo=bar&baz=qux
```

## Rationale

Service Key Parameters enable precise, context-aware data retrieval in ENS. For example, a text record can request the latest or historical ETH price from an oracle by specifying a block number parameter:

This approach allows clients to deterministically query external data sources (such as oracles) for time-specific or event-specific data, supporting advanced analytics, historical queries, and agentic systems that require verifiable context. Service Key Parameters make ENS a more powerful and flexible data registry for both human and machine consumers.

## Backwards Compatibility

Unaware clients will simply ignore the parameter and treat the full key as a string. Existing behavior is unaffected.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
