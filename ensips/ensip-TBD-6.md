---
ensip: TBD  
title: Hooks for Secure Onchain Data Resolution  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, Raffy.eth <@raffy@unruggable.com>  
created: 2024-10-07  
---

# Abstract 

This ENSIP introduces Hooks, a method for redirecting ENS records to a different resolver and chain ID. In the case of known resolvers, or a different resolver on the same chain, there are many benefits, including resolving secure records from a credential resolver, and caching the data location for later use. 

# Motivation

The goal of this ENSIP is to propose a new method for securely resolving onchain records. Hooks allow records to be redirected to known resolvers by specifying a resolver address and chain ID using ERC-7930 crosschain addresses. If the resolver is a known resolver, such as a credential resolver for a proof of personhood (PoP) or know your customer (KYC), it is possible to use a hook instead of a value in the ENS profile. The hook both notifies resolving clients of a credential, as well as provides the method for resolving the credential. 

# Specification

The key words "MUST", "MUST NOT", "REQUIRED", etc., are to be interpreted as described in RFC 2119.

## Using Hooks to Resolve ENS Records

Hooks are designed to securely resolve ENS records from resolvers with a specific address and chain ID (known resolver). The reason these resolvers are called known resolvers is because unlike traditional ENS resolution which resolves the value from whatever resolver the ENS name points to, with hooks, the clients become aware of the resolver address and chain ID before they either do or do not resolve the value, giving the client the opportunity to check whether or not the resolver is trusted. Resolvers can be any smart contract that implements the Extended Resolver interface (ENSIP-10) with a `resolve` method.

A hook comprises:

1. A single ABI-encoded ENS resolver function, such as `text(bytes32 node, string key)` or `contenthash(bytes32 node)`.

2. An ERC-7930 crosschain address. 

All resolvers MUST implement the `IExtendedResolver` interface specified in ENSIP-10. To call a known resolver, a client MUST call `resolve("", data)` where the `name` argument MUST be an empty string. The ENS name MAY be included in the key parameter, such as `"proof-of-person:maria.eth"`. The `data` field must contain the single ABI-encoded ENS resolver function call (as specified in ENSIP-1, ENSIP-5, etc), for example `text(node, key)`. The `resolve` function MUST either return valid return data for the specified function or `bytes("")` empty bytes. ERC-3668 offchain resolution SHOULD also be supported, such that a known resolver can resolve data offchain or from other chains, and have the results verified according to ERC-3668.  

### Hook Function using Bytes
```
function hook(
    string calldata ens-resolver-function,
    bytes erc-7930-crosschain-address-including-chain-id
) 
```

The bytes value of the function selector for `hook()` is `0x573ab61d`.

### Parameters

- **`ens-resolver-function`**: The function call to the resolver function (e.g., `text()`, `contenthash()`, `addr()`).
- **`erc-7930-crosschain-address-including-chain-id`**: The fully specified address and chain ID of the known resolver using ERC-7930 crosschain address format. 

## Using Hooks to Resolve ENS Records from an ENS Name.

A hook can either be resolved as the string version of the hook or as the ABI encoded calldata of the function, so that hooks can be saved as either a string value or bytes value such as for text() (ENSIP-5) records and data() (ENSIP-24) records.   

### Example: String Based
hook(
    "text(0x0f2efb96f8569aa24898732c1135c66ab581fa1ec6fab3af6dc411077b0858ac,'avatar')",
    0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8
);

### Example: ABI Encoded Calldata

Generate ABI Encoded Calldata:

```
cast calldata "hook(string,bytes)" "text(0x0f2efb96f8569aa24898732c1135c66ab581fa1ec6fab3af6dc411077b0858ac,'avatar')" "0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8"
```

ABI Encoded Calldata:

```
0x573ab61d000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c0000000000000000000000000000000000000000000000000000000000000005174657874283078306632656662393666383536396161323438393837333263313133356336366162353831666131656336666162336166366463343131303737623038353861632c276176617461722729000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001b00010000010114a94391031fe20f77d63af0b4f817dc4592b86ba80000000000
```

### Example: Resolving Credential From Known Credential Resolver

This example demonstrates how a client resolves a `proof-of-person` text record from a known credential resolver, verifying the resolver address before resolving the value.

**Scenario:**
- ENS Name: `maria.eth`
- Text Record Key: `proof-of-person`
- Expected Credential Value: `Maria Garcia, ID #1236234534`
- Known Credential Resolver Address: `0xa94391031FE20F77D63af0B4F817Dc4592b86BA8` (Ethereum Mainnet, Chain ID: 1)

**Step 1: Read the hook from the ENS name**

The client reads the hook from `maria.eth`'s text record (or data record) with key `proof-of-person`:

```
hook(
    "text(0x3cc095850df077d28e76eff1780be94210150f8133638973c65687be10fc9a83,'proof-of-person:maria.eth')",
    0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8
)
```

**Step 2: Parse the hook**

The client extracts:
- Resolver function: `text(bytes32,string)` with node `0x3cc095850df077d28e76eff1780be94210150f8133638973c65687be10fc9a83` (namehash of `maria.eth`) and key `'proof-of-person:maria.eth'`
- ERC-7930 address: `0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8` (27 bytes, Ethereum Mainnet format per [ERC-7930](https://eips.ethereum.org/EIPS/eip-7930))
  - Version: `0x0001` (2 bytes, decimal 1)
  - ChainType: `0x0001` (2 bytes, eip155 namespace)
  - ChainReferenceLength: `0x01` (1 byte, value 1)
  - ChainReference: `0x01` (1 byte, Ethereum Mainnet chain ID)
  - AddressLength: `0x14` (1 byte, value 20)
  - Address: `0xa94391031FE20F77D63af0B4F817Dc4592b86BA8` (20 bytes, resolver address)

**Step 3: Verify the resolver address**

Before resolving, the client verifies that the resolver address `0xa94391031FE20F77D63af0B4F817Dc4592b86BA8` matches a trusted credential resolver in their allowlist or registry. The client checks:
- Is this address a known, trusted credential resolver?
- Does the chain ID match the expected network?
- Should the client proceed with resolution?

**Step 4: Resolve the credential value**

If verification passes, the client calls the known resolver's `resolve` function with an empty string for the `name` parameter:

```solidity
resolve(
    "",
    abi.encodeWithSignature("text(bytes32,string)", 0x3cc095850df077d28e76eff1780be94210150f8133638973c65687be10fc9a83, "proof-of-person:maria.eth")
)
```

The resolver returns: `"Maria Garcia, ID #1236234534"`

**Step 5: Return the credential**

The client now has the verified credential value `Maria Garcia, ID #1236234534` that was resolved from the trusted credential resolver, ensuring the data source is authentic before displaying or using the credential.


## Rationale 

Hooks introduce redirection for resolving ENS records, which allows for resolving ENS records from "known" resolvers. Known resolvers may have security properties which are known, for example a resolver which resolves Proof-of-Personhood ID, or Know-your-Customer credentials. It has been a long felt need to securely resolve these types of records using ENS, and hooks() make it possible to leverage the properties of known smart contracts as resolvers to securely resolve credentials and secure records for ENS profiles. 

## Security Considerations

None.

## Backwards Compatibility

Hooks are backwards compatible; clients that are not aware of hooks, will simply resolve the hook() record as raw text or raw bytes. 

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).