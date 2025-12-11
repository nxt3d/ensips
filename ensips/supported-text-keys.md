---
title: Supported Text Keys
description: A standardized interface for discovering supported text record keys
contributors: 
    - premm.eth
    - clowes.eth
ensip:
  created: "2025-12-11"
  status: draft
---

# ENSIP-XX: Supported Text Keys

## Abstract

This ENSIP proposes a new resolver profile for discovering which text record keys are supported by a resolver for a given name.

## Motivation

ENS resolvers support arbitrary text records via ENSIP-5, allowing names to store key-value pairs such as email addresses, URLs, social media handles, and more. However, there is currently no standardized way to discover which text keys a resolver has data for without attempting to query each possible key.

This creates friction for:
- Applications that want to display all available text records for a name
- Wallets that need to show complete profile information
- Services that want to discover available metadata without prior knowledge of keys
- Developers building tools that work with ENS text records

By providing a standardized interface for text key discovery, we enable more efficient and complete ENS profile resolution.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Overview

This ENSIP introduces a new interface for discovering supported text record keys:

```solidity
/// @dev Interface selector: `0x92873114`
interface ISupportedTextKeys {
    /// @notice For a specific `node`, get an array of supported text keys.
    /// @param node The node (namehash).
    /// @return The text record keys for which the resolver has data.
    function supportedTextKeys(bytes32 node) external view returns (string[] memory);
}
```

The [ERC-165] identifier for this interface is `0x92873114`.

This function MUST return an array of `string` keys for which the resolver has text record data. If no text records are set, it MUST return an empty array.

### Extended Resolver Support

Resolvers implementing ENSIP-10 (Wildcard Resolution) MAY support this interface via extended resolution similar to how `text(bytes32 node, string key)` works.

The extended resolver callback for `supportedTextKeys` follows the same pattern as other resolver functions:

```solidity
interface IExtendedResolver {
    function resolve(bytes calldata name, bytes calldata data) external view returns (bytes memory);
}
```

When calling `supportedTextKeys` through an extended resolver:
1. The `name` parameter contains the DNS-encoded name
2. The `data` parameter contains the ABI-encoded call to `supportedTextKeys(bytes32)`
3. The resolver decodes the name and node, then returns the supported keys

### Rationale

The function signature mirrors `ISupportedDataKeys` from ENSIP-24, providing consistency across similar discovery interfaces.

The interface is kept simple and focused on the core use case: discovering what text keys are available for a given name.

### Example Implementation

#### Basic Resolver

```solidity
pragma solidity ^0.8.25;

contract Resolver {
    mapping(bytes32 => mapping(string => string)) private textRecords;
    mapping(bytes32 => string[]) private textKeys;
    
    function text(bytes32 node, string calldata key) external view returns (string memory) {
        return textRecords[node][key];
    }
    
    function supportedTextKeys(bytes32 node) external view returns (string[] memory) {
        return textKeys[node];
    }
}
```

#### Extended Resolver

```solidity
pragma solidity ^0.8.25;

contract ExtendedResolver {
    mapping(bytes32 => mapping(string => string)) private textRecords;
    mapping(bytes32 => string[]) private textKeys;
    
    function resolve(bytes calldata name, bytes calldata data) external view returns (bytes memory) {
        bytes32 node = namehash(name);
        bytes4 selector = bytes4(data[:4]);
        
        // Handle supportedTextKeys(bytes32)
        if (selector == 0x92873114) {
            return abi.encode(textKeys[node]);
        }
        
        // Handle text(bytes32,string)
        if (selector == 0x59d1d43c) {
            (, string memory key) = abi.decode(data[4:], (bytes32, string));
            return abi.encode(textRecords[node][key]);
        }
        
        revert("Unsupported function");
    }
}
```

### Usage Example

```javascript
// Query supported text keys
const resolver = await provider.getResolver('vitalik.eth');
const keys = await resolver.contract.supportedTextKeys(ethers.namehash('vitalik.eth'));

console.log(keys);
// ['email', 'url', 'avatar', 'com.twitter', 'com.github']

// Fetch all text records
const records = {};
for (const key of keys) {
    records[key] = await resolver.getText(key);
}
```

## Backwards Compatibility

This proposal introduces a new optional resolver profile and does not affect existing ENS functionality. Resolvers that do not implement this interface can continue to function normally.

```

## Security Considerations

### Gas Considerations

None. 

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

[ERC-165]: https://eips.ethereum.org/EIPS/eip-165

