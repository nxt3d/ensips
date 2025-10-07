---
title: Arbitrary Data Storage with Dedicated Data Record Type
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-09-25
---

## Abstract

This ENSIP proposes a new dedicated record type for storing arbitrary data in ENS names. This proposal introduces a purpose-built interface with the following function:

```
function data(bytes32 node, string calldata key) external view returns (bytes memory);
```

This dedicated approach provides a clean separation of concerns, avoiding conflicts with existing record types while offering a more intuitive interface for arbitrary data storage. The `key` parameter accepts string keys, providing a simple and developer-friendly interface for data storage.

## Motivation

ENS currently has a single dedicated record for storing multimedia content, the `contenthash` record (see [ENSIP-7](#)), which encodes web or content address data as a [multicodec](https://github.com/multiformats/multicodec). However, as new use cases emerge—particularly involving AI, where an ENS name may need to store rich contextual data, or AI protocol specific data hashes, there is a desire to use richer record types that can store unstructured binary data. While this ENSIP does not specify the types of data that may be used, it is possible, for example, to imagine multicodec records representing IPFS CIDs, URIs, and dataURLs, DIDs and hashed data commitments. 

While ENS does support `text` records ([ENSIP-5](#)), these are intended for human-readable text data and are limited to key-value string pairs. Address records support bytes, but using address records for arbitrary data might cause conflicts, and generally be confusing for developers.

By introducing a dedicated `data` record type, developers can store arbitrary key-value pairs without interfering with existing ENS functionality. For instance, a key like `"agent-context"` could be used to store context data as part of an AI agent, as pre-context for LLM prompts.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Overview

This ENSIP introduces a new record type for storing arbitrary data with the following interface:

```
data(bytes32 node, string calldata key) external view returns (bytes memory);
```

The `key` parameter accepts string keys, providing a simple and intuitive interface for data storage.

Resolvers implementing this ENSIP MUST emit the following event:

```
event DataChanged(bytes32 node, string indexed indexedKey, string key, bytes data);
```

### Example

Below is an illustrative snippet that shows how to set and retrieve arbitrary data:

```
pragma solidity ^0.8.0;

interface IDataResolver {
    event DataChanged(
        bytes32 indexed node,
        string indexed indexedKey,
        string key,
        bytes data
    );

    function data(
        bytes32 node,
        string calldata key
    ) external view returns (bytes memory);
}

contract Resolver is IDataResolver {
    mapping(bytes32 node => mapping(string key => bytes data)) private dataStore;
    
    function data(bytes32 node, string calldata key) external view returns (bytes memory) {
        return dataStore[node][key];
    }
    
    // setData function can be used to set the data (not shown)
}
```
Set and retrieve arbitrary data:

```
// Pseudo javascript example

// Store arbitrary data
const tx = await resolver.setData(node, "agent-context", "0x0001ABCD...");
await tx.wait();

// Retrieve arbitrary data
const result = await resolver.data(node, "agent-context");
```
### Rationale

ENS names have become widely used as identifiers across various applications and protocols, with growing demand for storing additional metadata, context data, and arbitrary data that existing record types cannot easily or efficiently accommodate. This ENSIP introduces a dedicated `data` record type using string-to-bytes key-value pairs, providing a simple and intuitive interface for data storage while ensuring the record type can adapt to various use cases including metadata storage and serialized data structures. This ENSIP intentionally leaves data encoding unspecified to accommodate future use cases.

## Backwards Compatibility

This proposal introduces a new record type and does not affect existing ENS functionality. It is fully backward compatible with all existing ENS records and resolvers.

## Security Considerations

None.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
