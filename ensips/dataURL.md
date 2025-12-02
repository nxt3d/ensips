---
title: Data URL Contenthash
description: Extends the contenthash field to support data URL content type using hooks
contributors: 
    - premm.eth
    - raffy.eth
    - "@RichardDwi"
ensip:
  created: "2024-06-07"
  status: Draft
---

# Abstract 

This ENSIP extends the `contenthash` field to support data URL content type, allowing content like webapps, images, and videos to be stored onchain.

# Motivation

The `contenthash` field has become the standard for using ENS names for decentralized websites and dapps. With ENSIP-10 and CCIP-Read (EIP-3668), resolving ENS records from L2s reduces the cost of using the `contenthash` field. This makes adopting the [data URL](https://datatracker.ietf.org/doc/html/rfc2397) standard feasible, allowing content like webapps, images, and videos to be stored onchain. This ENSIP introduces a new data URL content type for the `contenthash` field, enabling fully onchain websites without the need for hosting data offchain, for example when using IPFS. 

# Specification

[ENSIP-7](/7.md) introduced the `contenthash` field for resolving ENS names to content hosted on distributed systems such as IPFS and Swarm. The value returned by `contenthash` is represented as a machine-readable multicodec, which permits a wide range of protocols to be supported by ENS names. The format is specified as follows:

```
<protoCode uvarint><value []byte>
```

protoCodes and their meanings are specified in the [multiformats/multicodec](https://github.com/multiformats/multicodec) repository.

This ENSIP introduces a new multicodec, eth-calldata, which will be used for referencing the location of the Data URL using Hooks.

Until final protoCodes are approved the "Private Use Area" temporary codes should be used.

eth-calldata: 0x30009b

## Format

For Data URLs, we use ENSIP-XX Hooks to direct clients to a smart contract with a specified contract address, to resolve the data for the Data URL. 

The format of the hook is the abi encoded bytes (Ethereum calldata) of the function:

```
function hook(
    string calldata ens-resolver-function,
    bytes resolver-address
) 
```

Example for Vitalik.eth

```
function hook(
    "data(0xee6c4522aab0003e8d14cd40a6af439055fd2577951148c14b6cea9a53475835, 'data-url:vitalik.eth')",
    0xa94391031FE20F77D63af0B4F817Dc4592b86BA8
) 
```

- `0xee6c4...` is the `namehash` of 'vitalik.eth' the `node` of the ENS name

- `'data-url:vitalik.eth'` is the `key`, a string comprising: 

```
"data-url:" + <ENS Name>
```

- `resolver-address` – the address of the smart contract (resolver) where the data can be resolved on L1 Ethereum.

Format: `uvarint(codec2) + <ABI encoded 'hook' function calldata as bytes>`

## Web Gateway Resolution (e.g. eth.limo)

* The HTTP response MUST be a `200` OK.

* The HTTP response MUST be of `Content-type: $MIME`.

When resolving Data URLs, the URL of the request to the gateway is only used to determine the ENS name. Any path or query data of the request URL is ignored. For example `https://name.eth.limo` returns the same data URL as `https://name.eth.limo/a/b/c`. Single page applications (SPA) that are resolved in the browser may use the path information if necessary to modify the view of the SPA. 

### Optional Cache Management using AVID

DataURLs can stretch to the limit the amount of data that can be stored in a single blockchain record. Data can be in the kilobytes instead of the normal tens of bytes for standard blockchain records. For this reason it's desirable for gateways to be able to limit the number of resolutions of the DataURL data, especially when the data has not changed. For this reason the special key which AVID (ENSIP-XX) can be used `contenthash-hash`, to check to see if there have been any changes to the content. In the case of web gateway resolution, it is acceptable to display any content whether or not the `contenthash-hash` hash matches the hash of the final displayed content, because there is no standard way to display a verification check. For web gateway resolution the value of the `contenthash-hash` record can be used just as a "nonce" to see if the content has been updated.   

## Step by Step DataURL Resolution with Optional Cache Management using AVID. 

Web gateways MAY follow these steps to optimize RPC calls:

**Step 1: Check for `contenthash` value**

Resolve the `contenthash` record, including offchain resolution, ERC-3668, to check to see if the contenthash record is not empty and has the 0x30009b multicodec value. If it does not, the record is not a Data URL. 

**Step 2: Resolve the `contenthash-hash` data() record (ENSIP-24)**

Resolve the contenthash-hash `data()` record (ENSIP-24), which must support ERC-3668 offchain resolution. Compare the value of the record to a cached value, if the value is the same return the cached data as the web gateway response, else, proceed to step 3. 

**Step 3: Parse the Hook**

Parse the resolved `contenthash` value to extract the Hook. The Hook MUST be in the format specified in the Hooks specification, in the bytes format.

**Step 4: Resolve the full DataURL**

Extract the ABI-encoded hook calldata and resolve it according to the Hooks specification to fetch the full DataURL data from the `data-url:<ens-name>` record.


**Example:**

For `vitalik.eth`:
1. Gateway checks the `contenthash` record, and resolves the hook. 
2. Gateway resolves `data(node, "contenthash-hash")` from the vitalik.eth resolver, and compares it to its cached value. The value is not the same. 
3. Gateway resolves `contenthash` → receives hook calldata → resolves hook → receives full DataURL data.
4. Gateway caches nonce, and resolved contenthash bytes. 

# Rationale 

[ENSIP-7](/7.md) makes it possible to resolve contenthash records, allowing decentralized websites using decentralized storage such as IPFS and Swarm to be resolved using ENS names. With the addition of the Data URL contenthash type, it is possible to resolve a decentralized website that is fully onchain, avoiding the need for pinning data, for example, using IPFS.

An ENSIP was previously proposed by NameSys on the ENS DAO forum, [[Draft] ENSIP-17: DataURI Format in Contenthash](https://discuss.ens.domains/t/draft-ensip-17-datauri-format-in-contenthash/18048/7). Several methods for encoding that Data URL were discussed, including bypassing the multicodec and using the IPFS multicodec format among other methods. Adding a new protoCode was also discussed, and this ENSIP takes that approach to avoid overloading the top-level IPFS codec with other subtypes that aren't necessarily related to IPFS. Previously, a new data-url protoCode was proposed; however, it became necessary to separate the data URL contenthash and the onchain data, and this ENSIP takes the approach of using an Ethereum calldata protoCode with a special hook to resolve Data URLs.

# Security Considerations

Data URLs are intended for use in web browsers or other user-facing clients, so their security considerations are similar to any web application. However, onchain Data URLs can be safer than a traditional DNS website because the content can be stored entirely onchain, preventing attackers from altering or compromising the website.
  
# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


