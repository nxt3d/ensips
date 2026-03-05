---
title: ENS Record URI Scheme
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-08-18
---

# ENSIP-X: ENS Record URI Scheme

## Abstract

This ENSIP defines the `ensr:` URI scheme for addressing ENS records. It enables portable, unambiguous references to ENS records, including text records, addresses by coin type, contenthash, and arbitrary data records.

## Motivation

ENS names are widely used as human-readable identifiers, but clients lack a standard way to deep-link specific ENS records. `ensr:` makes records addressable in URIs so clients can point to the same data without ad hoc conventions. Default resolution is context specific and can vary by client; when deterministic behavior is required, use explicit queries such as `?text=`, `?addr=`, `?contenthash`, or `?data=`.

## Specification

### Summary

* Scheme: `ensr:`
* Authority: none
* Path: ENS name
* Queries:

  * `?text=<key>` to fetch a text record
  * `?addr[=<coinType>]` to fetch an address record; if `coinType` is omitted use 60 (ETH) as defined by ENSIP-11
  * `?contenthash` to fetch the contenthash explicitly
  * `?data=<key>` to fetch an arbitrary data record as defined by ENSIP-24
* Default when no query is given: resolution is context specific. In browser contexts clients often choose `contenthash`. In other contexts a designated text record may be preferred. Use explicit queries to avoid ambiguity.

### ABNF

ABNF follows RFC 3986 and RFC 5234; `unreserved` and `pct-encoded` are as defined in RFC 3986.

```
ensr-URI = "ensr:" ens-name [ "?" query ]

; ENS name (ASCII labels only) ; May be revisited in a future revision to include emoji and internationalized label support
ens-name = label ( "." label )
label = 1( ALPHA / DIGIT / "-" )

; Query: exactly one selector is allowed
query = addr-param / text-param / content-param / data-param

addr-param = "addr" [ "=" coin-type ]
coin-type = 1*DIGIT ; ENSIP-11 coin type (e.g., 60 for ETH)

text-param = "text=" text-key
text-key = 1*( unreserved / pct-encoded / ":" / "[" / "]" / %x2F )
; ":", "[", "]", "/" allowed for key parameters (ERC-8119)

content-param = "contenthash"

data-param = "data=" data-key
data-key = 1*( unreserved / pct-encoded / ":" / "[" / "]" / %x2F )
; ":", "[", "]", "/" allowed for key parameters (ERC-8119)

param = param-key "=" param-val
param-key = 1*( unreserved )
param-val = *( unreserved / pct-encoded )

```

### Resolution Semantics

1. Parse `ensr:` per ABNF.
2. Resolve `ens-name` via the ENS protocol.
3. If `?text=` is present, return the value for that text key.
4. If `?addr` is present with `=<coinType>`, return the corresponding address for that coin type. If `?addr` is present without a value, return the address for coin type 60 (ETH).
5. If `?contenthash` is present, return the contenthash value as defined by ENSIP-7.
6. If `?data=` is present, return the arbitrary data bytes for that key as defined by ENSIP-24.
7. If no query is present, resolution is context specific. Browser contexts commonly resolve to `contenthash`. Other contexts may select a designated text or data record. Use explicit queries to avoid ambiguity.

### Result Representation

This ENSIP defines identifiers, not transport. Implementations SHOULD return the raw record value directly, without mandatory JSON wrapping:

* Text records: return the UTF‑8 string exactly as stored. If the stored value happens to be JSON, clients will receive the JSON text unmodified.
* Address records: return the raw sequence, which may be context specific. In HTTP contexts this MAY be encoded as `application/octet-stream`. If text encoding is required, use `0x`‑prefixed hex.
* Contenthash: return the raw sequence, which may be context specific. In HTTP contexts this MAY be encoded as `application/octet-stream`. If text encoding is required, use `0x`‑prefixed hex.
* Data records: return the raw bytes sequence, which may be context specific. In HTTP contexts this MAY be encoded as `application/octet-stream`. If text encoding is required, use `0x`‑prefixed hex. 

No additional envelope or metadata is included at the URI resolution layer. Metadata such as encoding hints, or error codes SHOULD be conveyed by the transport protocol (e.g., HTTP headers or JSON‑RPC error objects) rather than by wrapping the value.

### Examples

```
ensr:neo.eth?text=eth.delegations:ens
ensr:neo.eth?text=com.twitter
ensr:neo.eth?text=agent-context
ensr:neo.eth?text=agent-registration[0x000100000101148004a169fb4a3325136eb29fa0ceb6d2e539a432][167] ; agent verification (ENSIP-25)

ensr:neo.eth?addr ; returns coin type 60 (ETH)
ensr:neo.eth?addr=60 ; ETH explicitly
ensr:neo.eth?addr=0 ; BTC

ensr:neo.eth?contenthash ; explicit website/content pointer

ensr:neo.eth?data=ai-context ; arbitrary data record

ensr:docs.eth ; context-specific default (not deterministic)
```

### Rationale

Clients already read ENS records, but linking to them lacks a simple, uniform URI standard. Using `?text`, `?addr`, `?contenthash`, and `?data` makes record specific resolution explicit and sharable in plain text contexts.

### Backwards Compatibility

Software that does not recognize `ensr:` will ignore these URIs. Existing ENS resolution behavior is unaffected.

### Security Considerations

None.

### Future Work

This revision restricts labels to ASCII. Internationalized labels, including emoji via punycode, may be specified in a future update. 

### References

* RFC 3986: Uniform Resource Identifier (URI): Generic Syntax
* RFC 5234: Augmented BNF for Syntax Specifications
* ERC-8119: Key Parameters
* ENSIP-11: Coin Type Definitions for ENS Address Resolution
* ENSIP-7: Contenthash for ENS
* ENSIP-24: Arbitrary Data Resolution
