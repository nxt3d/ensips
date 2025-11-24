---
title: Minimal Resolver Profiles
description: Overloading resolver profiles to allow resolution without requiring the node parameter
contributors: 
    - premm.eth
ensip:
  created: "2025-10-24"
  status: draft
---

# ENSIP: Minimal Resolver Profiles

## Abstract

This ENSIP proposes Minimal Resolver Profiles that allow resolution of records such as `text()`, `data()`, `addr()`, and `contenthash()` without requiring the `node` (namehash) parameter when using resolvers that implement [ENSIP-10](./10)'s `IExtendedResolver` interface. Since the `resolve(bytes calldata name, bytes calldata data)` function already receives the DNS-encoded name, the `node` parameter becomes redundant and can be omitted from the calldata.

## Motivation

Resolvers implementing [ENSIP-10](./10)'s `IExtendedResolver` interface receive the DNS-encoded name via the `name` parameter in the `resolve()` function. When resolving records such as `text()`, `data()`, `addr()`, and `contenthash()`, the standard resolver profiles require a `node` parameter (namehash) in the calldata, which is redundant since the resolver can derive the node from the `name` parameter or work directly with the name.

There is a clear need for Minimal Resolver Profiles that allow resolution without the `node` parameter when using `IExtendedResolver`. This would enable:

- Reduced gas costs by eliminating redundant namehash parameters in calldata
- Simplified calldata construction for clients using ENSIP-10 resolvers
- More efficient resolution when the name is already available in the `resolve()` call
- Support for resolvers that prefer working with names rather than namehashes

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Overview

This ENSIP introduces Minimal Resolver Profiles that allow `IExtendedResolver(resolver).resolve()` to handle calldata for the following record types without requiring the `node` parameter:

- Text records (as defined in [ENSIP-5](./5))
- Data records (as defined in [ENSIP-24](./24))
- Address records (as defined in [ENSIP-1](./1) and [ENSIP-9](./9))
- Contenthash records (as defined in [ENSIP-7](./7))

When a resolver implementing [ENSIP-10](./10)'s `IExtendedResolver` interface supports Minimal Resolver Profiles, it MUST implement the minimal resolver interfaces (`ITextResolverMin`, `IDataResolverMin`, `IAddrResolverMin`, `IAddressResolverMin`, `IContenthashResolverMin`) and declare support via ERC-165. Clients MUST check for interface support using `supportsInterface()` before constructing calldata for `text()`, `data()`, `addr()`, and `contenthash()` functions without including the `node` parameter.

The resolver MUST derive the node from the `name` parameter provided to `resolve()` or handle the resolution using the name directly.

Resolvers implementing Minimal Resolver Profiles MUST still support the corresponding standard resolver interfaces with the `node` parameter for backwards compatibility.

### Overloaded Text Resolver

When resolving text records via `IExtendedResolver(resolver).resolve()`, resolvers supporting overloaded text resolution MUST accept calldata for the following function signature:

```
/// @notice Returns the text data associated with a key (without requiring node parameter)
/// @param key A key to lookup text data for
/// @return The text data
function text(string calldata key) external view returns (string memory);
```

The resolver MUST derive the `node` from the `name` parameter provided to `resolve()` or handle the resolution using the name directly. The result MUST be the same as if `text(node, key)` were called with the `node` parameter corresponding to the name provided in the `resolve()` call.

Resolvers MUST still support the standard `text(bytes32 node, string calldata key)` function signature for backwards compatibility.

### Overloaded Data Resolver

When resolving data records via `IExtendedResolver(resolver).resolve()`, resolvers supporting overloaded data resolution MUST accept calldata for the following function signature:

```
/// @notice Get the data associated with the key (without requiring node parameter)
/// @param key The key
/// @return The associated arbitrary `bytes` data
function data(string calldata key) external view returns (bytes memory);
```

The resolver MUST derive the `node` from the `name` parameter provided to `resolve()` or handle the resolution using the name directly. The result MUST be the same as if `data(node, key)` were called with the `node` parameter corresponding to the name provided in the `resolve()` call.

Resolvers MUST still support the standard `data(bytes32 node, string calldata key)` function signature for backwards compatibility.

### Overloaded Address Resolver

When resolving address records via `IExtendedResolver(resolver).resolve()`, resolvers supporting overloaded address resolution MUST accept calldata for the following function signatures:

```
/// @notice Returns the Ethereum address (without requiring node parameter)
/// @return The Ethereum address
function addr() external view returns (address);

/// @notice Returns the address for the specified coin type (without requiring node parameter)
/// @param coinType The coin type index from SLIP44
/// @return The cryptocurrency address in its native binary format
function addr(uint256 coinType) external view returns (bytes memory);
```

The resolver MUST derive the `node` from the `name` parameter provided to `resolve()` or handle the resolution using the name directly. The result MUST be the same as if `addr(node)` or `addr(node, coinType)` were called with the `node` parameter corresponding to the name provided in the `resolve()` call.

Resolvers MUST still support the standard `addr(bytes32 node)` function signature from [ENSIP-1](./1) and `addr(bytes32 node, uint coinType)` function signature from [ENSIP-9](./9) for backwards compatibility.

### Overloaded Contenthash Resolver

When resolving contenthash records via `IExtendedResolver(resolver).resolve()`, resolvers supporting overloaded contenthash resolution MUST accept calldata for the following function signature:

```
/// @notice Returns the contenthash associated with this resolver (without requiring node parameter)
/// @return The contenthash data
function contenthash() external view returns (bytes memory);
```

The resolver MUST derive the `node` from the `name` parameter provided to `resolve()` or handle the resolution using the name directly. The result MUST be the same as if `contenthash(node)` were called with the `node` parameter corresponding to the name provided in the `resolve()` call.

Resolvers MUST still support the standard `contenthash(bytes32 node)` function signature for backwards compatibility.

### Minimal Resolver Interfaces

The following interfaces define the minimal resolver profiles, which include both the standard functions (with `node` parameter) and the minimal functions (without `node` parameter):

```
/// @dev Minimal text resolver interface (includes standard IERC634 functions)
interface ITextResolverMin {
    /// @notice Returns the text data associated with a key for an ENS name
    /// @param node A nodehash for an ENS name
    /// @param key A key to lookup text data for
    /// @return The text data
    function text(bytes32 node, string calldata key) external view returns (string memory);
    
    /// @notice Returns the text data associated with a key (without requiring node parameter)
    /// @param key A key to lookup text data for
    /// @return The text data
    function text(string calldata key) external view returns (string memory);
}

/// @dev Minimal data resolver interface (includes standard IDataResolver functions)
interface IDataResolverMin {
    /// @notice For a specific `node`, get the data associated with the key, `key`.
    /// @param node The node (namehash) for which data is being fetched.
    /// @param key The key.
    /// @return The associated arbitrary `bytes` data.
    function data(bytes32 node, string calldata key) external view returns (bytes memory);
    
    /// @notice Get the data associated with the key (without requiring node parameter)
    /// @param key The key
    /// @return The associated arbitrary `bytes` data
    function data(string calldata key) external view returns (bytes memory);
}

/// @dev Minimal address resolver interface (includes standard IAddrResolver functions)
interface IAddrResolverMin {
    /// @notice Returns the address associated with an ENS node.
    /// @param node The ENS node to query.
    /// @return The associated address.
    function addr(bytes32 node) external view returns (address);
    
    /// @notice Returns the Ethereum address (without requiring node parameter)
    /// @return The Ethereum address
    function addr() external view returns (address);
}

/// @dev Minimal address resolver interface with coin type (includes standard IAddressResolver functions)
interface IAddressResolverMin {
    /// @notice Returns the cryptocurrency address for the specified namehash and coin type.
    /// @param node A nodehash for an ENS name
    /// @param coinType The cryptocurrency coin type index from SLIP44
    /// @return The cryptocurrency address in its native binary format
    function addr(bytes32 node, uint256 coinType) external view returns (bytes memory);
    
    /// @notice Returns the address for the specified coin type (without requiring node parameter)
    /// @param coinType The coin type index from SLIP44
    /// @return The cryptocurrency address in its native binary format
    function addr(uint256 coinType) external view returns (bytes memory);
}

/// @dev Minimal contenthash resolver interface (includes standard IContenthashResolver functions)
interface IContenthashResolverMin {
    /// @notice Returns the contenthash associated with an ENS node.
    /// @param node The ENS node to query.
    /// @return The contenthash data.
    function contenthash(bytes32 node) external view returns (bytes memory);
    
    /// @notice Returns the contenthash associated with this resolver (without requiring node parameter)
    /// @return The contenthash data
    function contenthash() external view returns (bytes memory);
}
```

### ERC-165 Interface Support

Resolvers that implement [ENSIP-10](./10)'s `IExtendedResolver` interface and support Minimal Resolver Profiles MUST implement the minimal resolver interfaces (`ITextResolverMin`, `IDataResolverMin`, `IAddrResolverMin`, `IAddressResolverMin`, `IContenthashResolverMin`) and declare support via ERC-165's `supportsInterface()` function.

**Client Requirements:** Clients using `IExtendedResolver(resolver).resolve()` MUST check for interface support using `supportsInterface()` with the interface ID of the desired minimal resolver interface before constructing calldata without the `node` parameter. If the interface is not supported, clients MUST use the standard resolver function signatures with the `node` parameter in the calldata. Clients MUST NOT attempt to use Minimal Resolver Profiles if the interface is not supported.

### Rationale

When using `IExtendedResolver(resolver).resolve()`, the `name` parameter already contains the DNS-encoded name, making the `node` parameter redundant in the calldata. The resolver can derive the node from the name using the namehash algorithm, or work directly with the name if preferred.

Eliminating the `node` parameter from calldata reduces gas costs and simplifies calldata construction for clients. Since the name is already available in the `resolve()` call, there is no need to include the namehash in the calldata.

The requirement to support standard interfaces ensures backwards compatibility. Clients can always use the standard function signatures with the `node` parameter, while resolvers that support Minimal Resolver Profiles can be called more efficiently when interface support is detected.

ERC-165 interface checks allow clients to discover support for Minimal Resolver Profiles without attempting calls that may revert. Clients SHOULD check for interface support using `supportsInterface()` before constructing calldata without the `node` parameter.

### Example

Below is an illustrative snippet that shows an `IExtendedResolver` implementing Minimal Resolver Profiles:

```
pragma solidity ^0.8.25;

import "./IExtendedResolver.sol";  // ENSIP-10 ExtendedResolver
import "./ITextResolverMin.sol";  // Minimal text resolver (includes standard IERC634)
import "./IDataResolverMin.sol";  // Minimal data resolver (includes standard IDataResolver)
import "./IAddrResolverMin.sol";  // Minimal addr resolver (includes standard IAddrResolver)
import "./IAddressResolverMin.sol";  // Minimal address resolver (includes standard IAddressResolver)
import "./IContenthashResolverMin.sol";  // Minimal contenthash resolver (includes standard IContenthashResolver)

contract OverloadedResolver is IExtendedResolver, ITextResolverMin, IDataResolverMin, IAddrResolverMin, IAddressResolverMin, IContenthashResolverMin {
    uint constant private COIN_TYPE_ETH = 60;
    
    mapping(bytes32 => mapping(string => string)) private textStore;
    mapping(bytes32 => mapping(string => bytes)) private dataStore;
    mapping(bytes32 => mapping(uint => bytes)) private addrCoinStore;
    mapping(bytes32 => bytes) private contentHashStore;
    
    // Standard interfaces (required for backwards compatibility)
    function text(bytes32 node, string calldata key) external view returns (string memory) {
        return textStore[node][key];
    }
    
    function data(bytes32 node, string calldata key) external view returns (bytes memory) {
        return dataStore[node][key];
    }
    
    function addr(bytes32 node) external view returns (address) {
        bytes memory a = addrCoinStore[node][COIN_TYPE_ETH];
        if (a.length == 0) {
            return address(0);
        }
        return bytesToAddress(a);
    }
    
    function addr(bytes32 node, uint coinType) external view returns (bytes memory) {
        return addrCoinStore[node][coinType];
    }
    
    // Helper function to convert bytes to address
    function bytesToAddress(bytes memory b) private pure returns (address) {
        require(b.length == 20, "Invalid address length");
        address addr_;
        assembly {
            addr_ := mload(add(b, 20))
        }
        return addr_;
    }
    
    function contenthash(bytes32 node) external view returns (bytes memory) {
        return contentHashStore[node];
    }
    
    // Minimal resolver functions (without node parameter)
    // These require the resolver to be bound to a specific node via ExtendedResolver.resolve()
    function text(string calldata key) external view returns (string memory) {
        revert("Must be called via ExtendedResolver.resolve()");
    }
    
    function data(string calldata key) external view returns (bytes memory) {
        revert("Must be called via ExtendedResolver.resolve()");
    }
    
    function addr() external view returns (address) {
        revert("Must be called via ExtendedResolver.resolve()");
    }
    
    function addr(uint256 coinType) external view returns (bytes memory) {
        revert("Must be called via ExtendedResolver.resolve()");
    }
    
    function contenthash() external view returns (bytes memory) {
        revert("Must be called via ExtendedResolver.resolve()");
    }
    
    // ExtendedResolver.resolve() implementation
    function resolve(bytes calldata name, bytes calldata data) external view returns (bytes memory) {
        bytes32 node = namehash(name);
        
        // Check if calldata is for overloaded resolver functions (without node parameter)
        bytes4 sig = bytes4(data);
        
        if (sig == bytes4(keccak256("text(string)"))) {
            // Overloaded text: text(string)
            (string memory key) = abi.decode(data[4:], (string));
            return abi.encode(text(node, key));
        } else if (sig == bytes4(keccak256("data(string)"))) {
            // Overloaded data: data(string)
            (string memory key) = abi.decode(data[4:], (string));
            return abi.encode(data(node, key));
        } else if (sig == bytes4(keccak256("addr()"))) {
            // Overloaded addr: addr()
            return abi.encode(addr(node));
        } else if (sig == bytes4(keccak256("addr(uint256)"))) {
            // Overloaded addr: addr(uint)
            (uint coinType) = abi.decode(data[4:], (uint));
            return abi.encode(addr(node, coinType));
        } else if (sig == bytes4(keccak256("contenthash()"))) {
            // Overloaded contenthash: contenthash()
            return abi.encode(contenthash(node));
        }
        
        // Fallback to standard interfaces with node parameter
        // Standard text: text(bytes32, string)
        if (sig == bytes4(keccak256("text(bytes32,string)"))) {
            (bytes32 _node, string memory key) = abi.decode(data[4:], (bytes32, string));
            require(_node == node, "Node mismatch");
            return abi.encode(text(node, key));
        }
        // Standard data: data(bytes32, string)
        if (sig == bytes4(keccak256("data(bytes32,string)"))) {
            (bytes32 _node, string memory key) = abi.decode(data[4:], (bytes32, string));
            require(_node == node, "Node mismatch");
            return abi.encode(data(node, key));
        }
        // Standard addr: addr(bytes32)
        if (sig == bytes4(keccak256("addr(bytes32)"))) {
            (bytes32 _node) = abi.decode(data[4:], (bytes32));
            require(_node == node, "Node mismatch");
            return abi.encode(addr(node));
        }
        // Standard addr: addr(bytes32, uint)
        if (sig == bytes4(keccak256("addr(bytes32,uint256)"))) {
            (bytes32 _node, uint coinType) = abi.decode(data[4:], (bytes32, uint));
            require(_node == node, "Node mismatch");
            return abi.encode(addr(node, coinType));
        }
        // Standard contenthash: contenthash(bytes32)
        if (sig == bytes4(keccak256("contenthash(bytes32)"))) {
            (bytes32 _node) = abi.decode(data[4:], (bytes32));
            require(_node == node, "Node mismatch");
            return abi.encode(contenthash(node));
        }
        
        revert("Unsupported function");
    }
    
    // ERC-165 support
    function supportsInterface(bytes4 interfaceId) external pure returns (bool) {
        return interfaceId == type(IExtendedResolver).interfaceId ||
               interfaceId == type(ITextResolverMin).interfaceId ||
               interfaceId == type(IDataResolverMin).interfaceId ||
               interfaceId == type(IAddrResolverMin).interfaceId ||
               interfaceId == type(IAddressResolverMin).interfaceId ||
               interfaceId == type(IContenthashResolverMin).interfaceId ||
               interfaceId == 0x01ffc9a7; // ERC-165
    }
    
    // Note: This resolver MUST declare support for Minimal Resolver Profiles via
    // ERC-165 by implementing the minimal resolver interfaces and returning true
    // for supportsInterface() with the corresponding interface IDs. This allows
    // clients to discover support for Minimal Resolver Profiles.
}
```

Usage example:

```
// Pseudo code example

import { getEnsText, getEnsAddress, getEnsName } from '@library/ens';

const name = "example.eth";

// Minimal resolution - ENS functions handle interface support checks automatically
// When resolver supports Minimal Resolver Profiles, node parameter is omitted

// Get text record (no node parameter needed)
const text = await getEnsText(publicClient, { 
  name, 
  key: 'url' 
});

// Get data record (no node parameter needed)
const data = await getEnsData(publicClient, { 
  name, 
  key: 'agent-context' 
});

// Get Ethereum address (no node parameter needed)
const addr = await getEnsAddress(publicClient, { 
  name 
});

// Get address for specific coin type (no node parameter needed)
const addrCoin = await getEnsAddress(publicClient, { 
  name, 
  coinType: 60 // Ethereum
});

// Get contenthash (no node parameter needed)
const contenthash = await getEnsContentHash(publicClient, { 
  name 
});

// Standard resolution (always works) - node is handled internally by ENS utilities
const textStandard = await getEnsText(publicClient, { 
  name, 
  key: 'url' 
});
```

## Backwards Compatibility

This proposal introduces Minimal Resolver Profiles for use with `IExtendedResolver(resolver).resolve()`. All resolvers implementing Minimal Resolver Profiles MUST implement the minimal resolver interfaces and declare support via ERC-165, and MUST also support the corresponding standard resolver interfaces with the `node` parameter, ensuring full backwards compatibility.

Clients unaware of Minimal Resolver Profiles will continue to work using standard resolver function signatures with the `node` parameter in the calldata. Clients that detect support for minimal resolver interfaces using ERC-165 can construct calldata without the `node` parameter, reducing gas costs. Clients MUST check for interface support using `supportsInterface()` before attempting to use Minimal Resolver Profiles.

## Security Considerations

Resolvers implementing Minimal Resolver Profiles SHOULD validate that when handling standard interface calls (with the `node` parameter), the `node` parameter matches the node derived from the `name` parameter provided to `resolve()`. This prevents potential confusion or misuse where calldata contains a different node than the name being resolved.

ERC-165 interface checks allow clients to safely discover support for Minimal Resolver Profiles without constructing calldata that may cause the resolver to revert.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

[ERC-165]: https://eips.ethereum.org/EIPS/eip-165

