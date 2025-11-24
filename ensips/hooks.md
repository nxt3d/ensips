---
title: Hooks for Secure Onchain Data Resolution
description: A method for redirecting ENS records to a different resolver and chain ID
contributors: 
    - premm.eth
    - raffy.eth
ensip:
  created: "2024-10-07"
  status: draft
---

# Abstract 

This ENSIP introduces Hooks, a method for redirecting ENS records to a different resolver, possibly on a different chain. In the case of known resolvers there are many benefits, including resolving secure records. It can also be a benefit for clients who want to resolve the data location of data before resolving the data.

# Motivation

The goal of this ENSIP is to propose a new method for securely resolving onchain records. Hooks allow records to be redirected to known resolvers by specifying a resolver address and chain ID using ERC-7930 crosschain addresses. If the resolver is a known resolver, such as a credential resolver for a proof of personhood (PoP) or know your customer (KYC), it is possible to use a hook instead of a value in the ENS profile. The hook both notifies resolving clients of a credential, as well as provides the method for resolving the credential. 

# Specification

The key words "MUST", "MUST NOT", "REQUIRED", etc., are to be interpreted as described in RFC 2119.

## Using Hooks to Resolve ENS Records

Hooks are designed to securely resolve ENS records from resolvers with a specific address and chain ID (known resolver). The reason these resolvers are called known resolvers is because unlike traditional ENS resolution which resolves the value from whatever resolver the ENS name points to, with hooks, the clients become aware of the resolver address and chain ID before resolving the value, giving the client the opportunity to check whether or not the resolver is trusted. Resolvers can be any smart contract that implements the Extended Resolver interface (ENSIP-10) with a `resolve` method.

A hook comprises:

1. A single ENS resolver function, such as `text(bytes32 node, string key)` [ENSIP-5](/5.md), `data(bytes32 node, string key)`[ENSIP-24](/24.md) or `contenthash(bytes32 node)` [ENSIP-7](/7.md).

2. An ERC-7930 crosschain address. 

All resolvers MUST implement the `IExtendedResolver` interface specified in ENSIP-10. To call a known resolver, a client MUST call the `resolve(bytes memory name, bytes memory data)` where the `name` argument MUST be the ENS name if known, if not known MAY be left blank. The ENS name CAN be included in key parameters, such as `"proof-of-person:maria.eth"`, allowing for hooks to be able to be resolved without context. It is also possible to use the `multicall()` [ENSIP-23](/23.md) function to resolve multiple records in a single call. ERC-3668 offchain resolution MUST also be supported, such that a known resolver can resolve data offchain or from other chains, and have the results verified according to ERC-3668.  

### Hook Function
```
function hook(
    string calldata ens-resolver-function,
    bytes erc-7930-crosschain-address-including-chain-id
) 
```

The bytes value of the function selector for `hook()` is `0x573ab61d`.

### Parameters

- **`ens-resolver-function`**: The function call to the resolver function (e.g., `text()`, `contenthash()`, `addr()`).
- **`erc-7930-crosschain-address-including-chain-id`**: The fully specified address and chain ID of the known resolver using ERC-7930 crosschain address format in bytes. 

## Using Hooks to Resolve ENS Records from an ENS Name.

A hook can either be resolved as the string version of the hook or as the ABI encoded calldata of the function, so that hooks can be saved as either a string value or bytes value such as for text() (ENSIP-5) records and data() (ENSIP-24) records.   

### Steps for Resolving a Hook

1. **Get the hook** from the ENS name's text or data record
2. **Parse the hook** to extract the resolver function and ERC-7930 address
3. **Verify the resolver address** to ensure it matches a trusted resolver (optional but recommended)
4. **Resolve the credential value** by calling the known resolver's `resolve()` function
5. **Return the credential** value for use by the client

### Example: String Based

The string based hook should be formatted using hex encoding defined in RFC 4648 for binary values such as the `namehash` and `ERC-7930` address, Section 8, including the 0x prefixs and using single quotes around strings parameters. Newlines should not be included, and the `;` at the end of the hook command SHOULD not be included. 

hook("text(0x0f2efb96f8569aa24898732c1135c66ab581fa1ec6fab3af6dc411077b0858ac,'avatar')", 0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8)

### Example: Resolving Credential From Known Credential Resolver

This example demonstrates how a client resolves a `proof-of-person` text record from a known resolver, verifying the resolver address before resolving the value.

**Scenario:**
- ENS Name: `maria.eth`
- Known Resolver Text Record Key: `proof-of-person:maria.eth`
- Expected Credential Value: `PoP ID #1236234534`
- Known Credential Resolver ERC-7930 Address: `0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8` (Ethereum Mainnet, Chain ID: 1)

**Step 1: Read the hook from the ENS name**

The client reads the hook from `maria.eth`'s text record with key `proof-of-person` and the key parameter `maria.eth`:

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

**Step 3: Verify the resolver address**

Before resolving, the client verifies that the ERC-7930 resolver address `0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8` matches a trusted credential resolver. A third party registry can be used to check that the resolver is trusted and belongs to a known credential provider. The client checks:
- Is this address a known, trusted resolver?
- Does the chain ID match the expected network?
- Should the client proceed with resolution?

**Step 4: Resolve the credential value**

If verification passes, the client calls the known resolver's `resolve` function:

```solidity
resolve(
    0x056d617269610365746800, // DNS encoded maria.eth
    abi.encodeWithSignature("text(bytes32,string)", 0x3cc095850df077d28e76eff1780be94210150f8133638973c65687be10fc9a83, "proof-of-person:maria.eth")
)
```

The resolver returns: `"Pop ID #1236234534"`

**Step 5: Return the credential**

The client now has the verified credential value `Pop ID #1236234534` that was resolved from the trusted credential resolver, ensuring the data source is authentic before displaying or using the credential.

## Rationale 

Hooks introduce redirection for resolving ENS records, which allows for resolving ENS records from "known" resolvers. Known resolvers may have security properties which are known, for example a resolver which resolves Proof-of-Personhood ID, or Know-your-Customer credentials. It has been a long felt need to securely resolve these types of records using ENS, and hooks() make it possible to leverage the properties of known smart contracts as resolvers to securely resolve credentials and secure records for ENS profiles. 

## Security Considerations

None.

## Backwards Compatibility

Hooks are backwards compatible; clients that are not aware of hooks, will simply resolve the hook() record as raw text or raw bytes. 

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).