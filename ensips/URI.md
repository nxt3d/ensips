---
title: URI Contenthash
description: Extends the contenthash field to support URI content type
contributors: 
    - premm.eth
    - raffy.eth
    - "@RichardDwi"
ensip:
  created: "2024-06-07"
  status: Draft
---

# Abstract 

This ENSIP extends the `contenthash` field to support the URI content type, allowing browsers to redirect to a standard URI when loading an ENS name.

# Motivation

The `contenthash` field has become the standard for using ENS names for decentralized websites and dapps. With ENSIP-10 and CCIP-Read (EIP-3668), resolving ENS records from L2s reduces the cost of using the `contenthash` field. This ENSIP introduces a new URI content type for the `contenthash` field, allowing browsers to redirect to a standard URI when loading an ENS name. While URIs, such as ethereum.org, are not decentralized or onchain, this makes ENS names more backward compatible with Web2 and is a convenience for users. 

# Specification

[ENSIP-7](/7.md) introduced the `contenthash` field for resolving ENS names to content hosted on distributed systems such as IPFS. The value returned by `contenthash` is represented as a machine-readable multicodec, which permits a wide range of protocols to be supported by ENS names. The format is specified as follows:

```
<protoCode uvarint><value []byte>
```

ProtoCodes and their meanings are specified in the [multiformats/multicodec](https://github.com/multiformats/multicodec) repository.

This ENSIP introduces a new multicodec,`uri`, for the `contenthash` field.

Until final protoCodes are approved, the "Private Use Area" temporary codes should be used.

uri: 0x3000f2

## Format

Format: `uvarint(codec) + <URI as utf8 bytes>`

## Web Gateway Resolution (e.g. eth.limo)

* The HTTP response MUST be a `307` Temporary Redirect.
	
* The response `Location` MUST be `$URI`, e.g. https://domain.com/a/b.c?d=e.

A reasonable limit may be placed by clients on the number of characters in the URI, but at least 256 bytes of UTF-8 characters should be supported.

# Rationale 

[ENSIP-7](/7.md) makes it possible to resolve contenthash records, allowing decentralized websites using decentralized storage such as IPFS to be resolved using ENS names. Many users, however, would prefer to simply redirect their ENS name to a URI. It is currently possible to include a URI in the text record 'url'; however, this has traditionally been used as a profile record to link to the user's website, for example, a blog or homepage. This ENSIP makes it possible to redirect the ENS name to a URI using the contenthash field, intended for resolving within the web browser.

# Security Considerations

URIs are intended for use in web browsers or other user-facing clients, so their security considerations are similar to those of any web application. URIs redirect to external websites, so users should be aware that the content is not stored onchain and may be changed by the website owner.
  
# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


