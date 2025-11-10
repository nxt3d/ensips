---
title: Data URL and URI Contenthash
description: Extends the contenthash field to support data URL and URI content types
contributors: 
    - premm.eth
    - raffy.eth
    - "@RichardDwi"
ensip:
  created: "2024-06-07"
  status: Draft
---

# Abstract 

This ENSIP extends the `contenthash` field to support two additional content types: data URL and URI.

# Motivation

The `contenthash` field has become the standard for using ENS names for decentralized websites and dapps. With ENSIP-10 and CCIP-Read (EIP-3668), resolving ENS records from L2s reduces the cost of using the `contenthash` field. This makes adopting the [data URL](https://datatracker.ietf.org/doc/html/rfc2397) standard feasible, allowing content like webapps, images, and videos to be stored onchain. This ENSIP also introduces a new URI content type for the `contenthash` field, allowing browsers to redirect to a standard URI when loading an ENS name. While URIs, such as ethereum.org, are not decentralized or onchain, it makes ENS names more reverse compatible with Web2 and is a convenience for users. 

# Specification

[ENSIP-7](/7.md) introduced the `contenthash` field for resolving ENS names to content hosted on distributed systems such as IPFS and Swarm. The value returned by `contenthash` is represented as a machine-readable multicodec, which permits a wide range of protocols to be supported by ENS names. The format is specified as follows:

```
<protoCode uvarint><value []byte>
```

protoCodes and their meanings are specified in the [multiformats/multicodec](https://github.com/multiformats/multicodec) repository.

This ENSIP introduces two new types of multicodecs, uri and eth-calldata (which will be used for referencing the location of the Data URL using Hooks).  

Until final protoCodes are approved the "Private Use Area" temporary codes should be used.

uri: 0x3000f2

eth-calldata: 0x30009b

## New Formats 
**URI**

Format: `uvarint(codec1) + <URI as utf8 bytes>`

**Data URL**

For Data URLs, we use ENSIP-XX Hooks to direct clients to a smart contract with a specified contract address and coinType (chain id), using ERC-7930 binary addresses, to resolve the data for the Data URL. 

The format of the hook is the abi encoded bytes (Ethereum calldata) of the function:

```
function hook(
    string calldata ens-resolver-function,
    bytes resolver-address-including-chain-id
) 
```

Example for Vitalik.eth

```
function hook(
    "data(0xee6c4522aab0003e8d14cd40a6af439055fd2577951148c14b6cea9a53475835, 'data-url:vitalik.eth')",
    0x00010000010114a94391031FE20F77D63af0B4F817Dc4592b86BA8
) 
```

- `0xee6c4...` is the `namehash` of 'vitalik.eth' the `node` of the ENS name

- `'data-url:vitalik.eth'` is the `key`, a string comprising: 

```
"data-url:" + <ENS Name>
```

- `resolver-address-including-chain-id` – the ERC-7930 binary address of the smart contract (resolver) where the data can be resolved

Format: `uvarint(codec2) + <ABI encoded 'hook' function calldata as bytes>`

## Web Gateway Resolution (e.g. eth.limo)

**URI:** 

* The HTTP response MUST be a `307` Temporary Redirect.
	
* The response `Location` MUST be `$URI` e.g. https://domain.com/a/b.c?d=e.

A reasonable limit may be placed by clients on the number of characters in the URI, but at least 256 bytes of UTF-8 characters should be supported. 

**Data URL:**

* The HTTP response MUST be a `200` OK.

* The HTTP response MUST be of `Content-type: $MIME`.

When resolving Data URLs, the URL of the request to the gateway is only used to determine the ENS name. Any path or query data of the request URL is ignored. For example `https://name.eth.limo` returns the same data URL as `https://name.eth.limo/a/b/c`. Single page applications (SPA) that are resolved in the browser may use the path information if necessary to modify the view of the SPA. 

### Cache Management using Nonces

DataURLs can stretch to the limit the amount of data that can be stored in a single blockchain record. Data can be in the kilobytes instead of the normal tens of bytes for standard blockchain records. For this reason it's desirable for gateways to be able to limit the number of resolutions of the DataURL data, especially when the data has not changed. For this reason an optional `contenthash-nonce` data record has been introduced to allow gateways to reduce the number of resolutions of the full dataURL data. 

It is also desirable that the nonce value be able to be read from, for example, the same contract where the dataURL is stored. For this reason the `contenthash-nonce` record value MUST be a hook, in binary format. The multicodec format should not be used as this is only necessary for the overloaded contenthash record.

Web gateways may check the `contenthash-nonce` `data()` [ENSIP-24](/24.md) record to get the Hook. For backwards compatibility, the `contenthash-nonce` text record can also be used with the string format of a Hook. See the Hooks specification for the bytes and string formats. At the time of this writing, ENSIP-24 has been newly introduced, therefore using the `contenthash-nonce` text record as a backup is needed, however, in the future it may be deprecated when the support of ENSIP-24 is sufficiently well adopted. 

If there is a properly formatted Hook value for the `contenthash-nonce`, web gateways MUST resolve the data of the hook resolver contract, and fetch the data from a data() record called `data-url-nonce`. The value should be a bytes number. A web gateway may not reload the full dataURL when a resolution request is made, if the nonce value in `data-url-nonce` has not changed. 

## Step by Step DataURL Resolution with Optional Nonce Checking. 

A step-by-step example of resolving a DataURL with nonce checking. Web gateways SHOULD follow these steps to optimize caching:

**Step 1: Check for cached nonce value**

If the gateway has a previously cached nonce value for this ENS name, proceed to Step 2. If no cached nonce exists, proceed to Step 5 to resolve the full DataURL.

**Step 2: Resolve the `contenthash-nonce` record**

Attempt to read the `contenthash-nonce` record using `data(node, "contenthash-nonce")` [ENSIP-24](/24.md). If the record returns empty bytes (`""`) or ENSIP-24 is not supported, fall back to reading the `contenthash-nonce` text record using `text(node, "contenthash-nonce")` [ENSIP-5](/5.md).

**Step 3: Parse the Hook**

Parse the resolved `contenthash-nonce` value to extract the Hook. The Hook MUST be in the format specified in the Hooks specification, either as bytes (from `data()` record) or as a string (from `text()` record).

**Step 4: Resolve the current nonce value**

Using the Hook from Step 3, resolve the hook resolver contract to fetch the `data-url-nonce` record. Extract the nonce value as bytes. Web gateways MAY stop without resolving the DataURL if the hook resolver contract specified is not on L1 Ethereum.

**Step 5: Compare nonce values**

Compare the current nonce value from Step 4 with the cached nonce value from Step 1:
- If the values match AND the gateway has a cached DataURL, return the cached DataURL without resolving the dataURL from the hook resolver.
- If the values differ OR no cached nonce exists, proceed to Step 6. If no nonce exists, web gateways MAY manage caching of DataURLs using their own system, for example only resolving the dataURL from the hook contract after a certain amount of time has passed after a successful resolution. 

**Step 6: Resolve the full DataURL**

Resolve the `contenthash` record for the ENS name. Parse the multicodec to identify it as `eth-calldata` (codec `0x30009b`). Extract the ABI-encoded hook calldata and resolve it according to the Hooks specification to fetch the full DataURL data from the `data-url:<ens-name>` record. Web gateways MAY stop without resolving the DataURL if the hook resolver contract specified in the hook is not on L1 Ethereum.

**Step 7: Cache the nonce and DataURL**

Store both the current nonce value (from Step 4) and the resolved DataURL data (from Step 6) in the gateway's cache, associated with the ENS name. This allows future requests to skip the full DataURL resolution if the nonce has not changed.

**Example:**

For `vitalik.eth`:
1. Gateway checks cache: no cached nonce found.
2. Gateway resolves `data(node, "contenthash-nonce")` → receives Hook bytes.
3. Gateway parses Hook: `hook("data(0xee6c45..., 'data-url-nonce:vitalik.eth')", 0x0001...)`
4. Gateway resolves hook → receives `data-url-nonce:vitalik.eth` value: `0x0000000000000000000000000000000000000000000000000000000000000001`
5. Nonce was previously 0, proceed to full DataURL.
6. Gateway resolves `contenthash` → receives hook calldata → resolves hook → receives full DataURL data.
7. Gateway caches nonce `0x0000000000000000000000000000000000000000000000000000000000000001` and DataURL data.

On subsequent requests:
1. Gateway checks cache: finds cached nonce `0x0000000000000000000000000000000000000000000000000000000000000001`.
2-4. Gateway resolves current nonce: `0x0000000000000000000000000000000000000000000000000000000000000001`.
5. Values match → return cached DataURL without resolving the full hook.
6-7. Skip full resolution and caching.

# Rationale 

[ENSIP-7](/7.md) makes it possible to resolve contenthash records, allowing decentralized websites using decentralized storage such as IPFS and Swarm to be resolved using ENS names. Many users, however, would prefer to simply redirect their ENS name to a URI. It is currently possible to include a URI in the text record 'url'; however, this has traditionally been used as a profile record to link to a website of the user, for example, a blog or homepage. This ENSIP makes it possible to redirect the ENS name to a URI using the contenthash field, intended for resolving within the web browser. With the addition of the Data URL contenthash type, it is possible to resolve a decentralized website that is fully onchain, avoiding the need for pinning data, for example, using IPFS.

# Security Considerations

Data URLs and URIs are intended for use in web browsers or other user-facing clients, so their security considerations are similar to any web application. However, onchain Data URLs can be safer than a traditional DNS website because the content can be stored entirely onchain, preventing attackers from altering or compromising the website.
  
# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


