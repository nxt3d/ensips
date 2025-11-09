---
title: Agentic Verifiable and Inspectable Data (AVID)
description: A simple verification protocol for agent workflows using ENS data records
contributors:
  - premm.eth
ensip:
  created: "2025-01-XX"
  status: draft
---

# ENSIP-TBD: Agentic Verifiable and Inspectable Data (AVID)

## Abstract

This ENSIP defines AVID (Agentic Verifiable and Inspectable Data), a simple verification protocol for agent workflows. Any data blob can declare its source and a hash. A client verifies byte-for-byte integrity by comparing a locally computed hash to a hash stored under an ENS name. The MIME type of the data can also be declared, with `text/plain; charset=utf-8` being the default. 

## Motivation

There is a need for a lightweight, deterministic way to verify data integrity in agent workflows. Current solutions often require complex signature schemes or external attestation services. IPFS-based solutions require using the IPFS network, which may not be desired. AVID provides a minimal protocol that:

- Makes arbitrary data verifiable by agents with minimal ceremony
- Binds data integrity to an ENS name that is easy to resolve across chains
- Defines a precise hashing rule so results are deterministic

AVID does not define a new transport it use ENS and does not solve authenticity by itself (authenticity comes from control of the ENS name).

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Terminology

**Blob.** The exact bytes of a resource as fetched by the client.

**Source.** An ENS name that anchors the authoritative hash. Data MUST be resolved via ENS using the `ensr:` URI scheme as defined in the ENS Record URI Scheme specification.

**AVID tag.** A wrapper that declares where to look and optionally the MIME type of the data.

**Resolver.** Any ENS resolver associated with an ENS name.

### Hashing Rule

Clients MUST hash exactly the bytes from the first byte to the last byte of the blob. No normalization, no line ending changes, no whitespace trimming. Encode data as received. The hash algorithm is determined from the method byte in the hash record (see Hash Format section). The default method is `0x00` (keccak256).

### Hash Format

Hash records MUST be stored in a self-describing format that includes the hash algorithm identifier:

**Format:** `[method: 1 byte][length: 1 byte][hash: up to 29 bytes]`

* **Method byte (1 byte):** Hash algorithm identifier
  * `0x00` = keccak256
  * Other values reserved for future algorithms
* **Length byte (1 byte):** Length of the hash in bytes (MUST be 1-29). If the actual hash length exceeds 29 bytes, only the first 29 bytes are stored.
* **Hash bytes (1-29 bytes):** The first 29 bytes of the hash, or the full hash if it is 29 bytes or less

This format allows hash records to be self-describing, eliminating the need to declare the hash method elsewhere. The maximum total size is 31 bytes (1 + 1 + 29), which fits within a single storage slot.

### ENS Storage

The hash and data MUST be stored under the ENS name referenced by the source using ENSIP-24 `data()` records.

When using `ensr:` URIs, the `?data=` parameter MUST include the full key with the `-data` suffix. The hash key is automatically derived by AVID by replacing `-data` with `-hash` in the key.

* `{label-data}`: raw bytes of the actual data payload (e.g., `blob-data`, `context-data`)
* `{label-hash}`: prefixed hash format as defined above (method byte + length byte + hash bytes), derived by replacing `-data` with `-hash` in `{label-data}` (e.g., `blob-hash`, `context-hash`)

If no `?data=` parameter is specified in the `ensr:` URI, the default key is `"blob-data"`, which results in:
* `blob-data`: the actual data bytes
* `blob-hash`: the hash of the data

For example, with key `"context-data"` in the `ensr:` URI:
* `context-data`: the actual data bytes
* `context-hash`: the hash of the data (derived by replacing `-data` with `-hash`)

### AVID TAGS

AVID MAY be embedded as an HTML/XML tag:

```
<AVID SRC="ensr:somedata.data.eth?data=data-blob-data" MIMETYPE="text/plain; charset=utf-8" />
```

The `SRC` attribute MUST contain an `ensr:` URI. The `MIMETYPE` attribute is OPTIONAL and specifies the MIME type of the data (default: `text/plain; charset=utf-8`).

Clients MUST resolve data using the `ensr:` URI from `SRC` to fetch the blob bytes from ENS via `data(node, "{label-data}")` where `{label-data}` is the full key including `-data` suffix, extract bytes exactly as received, derive the hash key by replacing `-data` with `-hash`, read the hash record from `data(node, "{label-hash}")` to determine the algorithm, compute the digest using that algorithm, and compare.

### AVID in STRUCTURED DATA

AVID metadata MAY be represented as a JSON object:

```
{
  "type": "static-resource",
  "avid": {
    "source": "ensr:somedata.data.eth?data=data-blob-data",
    "mimeType": "text/plain; charset=utf-8"
  }
}
```

The `avid` field MUST be an object containing:
* `source` (REQUIRED): An `ensr:` URI specifying the data location
* `mimeType` (OPTIONAL): The MIME type of the data (default: `text/plain; charset=utf-8`)

Verification steps are identical. Extract the `ensr:` URI from the `avid.source` field, resolve data via that URI (which reads `data(node, "{label-data}")` where `node` is the namehash of the full ENS name (e.g. "somedata.data.eth") and `{label-data}` includes the `-data` suffix), derive the hash key by replacing `-data` with `-hash`, read the hash record from `data(node, "{label-hash}")` to determine the algorithm, fetch bytes from ENS, hash using the determined algorithm, and compare.

### ENSR URI Scheme

The `ensr:` URI scheme as defined in the ENS Record URI Scheme specification has context-specific default behavior when no query parameters are provided. For AVID, the default behavior is explicitly defined: when an `ensr:` URI is provided without a `?data=` parameter, clients MUST resolve the `data(node, "blob-data")` record, where `node` is the namehash of the ENS name. This ensures deterministic behavior for AVID verification regardless of context.

When using `ensr:` URIs with AVID, the `?data=` parameter MUST include the full key with the `-data` suffix. The hash key is derived by replacing `-data` with `-hash` in the key.

For example:
* `ensr:docs.data.eth` (no query) → resolves to `data(node, "blob-data")`, hash at `data(node, "blob-hash")`
* `ensr:docs.data.eth?data=context-data` → resolves to `data(node, "context-data")`, hash at `data(node, "context-hash")`

### Verification Flow

Clients MUST follow these steps for verification:

1. Parse the source `ensr:` URI to extract the ENS name and data key. The data key in the `?data=` parameter MUST include the `-data` suffix (e.g., `blob-data`, `context-data`). If no `?data=` parameter is provided, use `"blob-data"` as the default key.
2. Derive the hash key by replacing `-data` with `-hash` in the data key (e.g., `blob-data` → `blob-hash`, `context-data` → `context-hash`).
3. Resolve `SRC` ENS name.
4. Resolve data using the `ensr:` URI from the source. This retrieves the raw bytes from the ENS name's `data(node, "{label-data}")` record, where `node` is the namehash of the ENS name and `{label-data}` is the full key including `-data` suffix. The data is now available for use.
5. Fetch the blob bytes from the resolved ENS data.
6. Read ENSIP-24 `data(node, "{label-hash}")` where `node` is the namehash of the ENS name and `{label-hash}` is the derived hash key.
7. Parse the hash record to extract the method byte, length byte, and hash bytes.
8. Determine algorithm from the method byte (e.g., `0x00` = keccak256). If the method is unsupported, reject gracefully.
9. Compute digest of exact bytes using the algorithm determined from step 8.
10. If the length from the hash record is greater than 29, reject with an error. Otherwise, compare the computed digest (first N bytes where N = length from hash record) with the hash bytes from the hash record. Match = success, mismatch = fail.

## Rationale

AVID provides a minimal, deterministic verification protocol that leverages existing ENS infrastructure. By using ENS data records and the `ensr:` URI scheme, AVID avoids the need for external URLs or complex transport mechanisms. The protocol focuses solely on integrity verification, leaving authenticity to ENS name control.

The use of label-based records (`{label}-data` and `{label}-hash`) allows multiple data blobs to be stored under a single ENS name, enabling efficient organization and discovery. This approach extends the same trust and authenticity of the ENS name to all data records stored under it.

## Examples

### HTML TAG

```
<AVID SRC="ensr:docs.data.eth?data=context-data" MIMETYPE="application/json" />
```

### JSON Metadata

```
{
  "post": {
    "title": "Hello",
    "body": "..."
  },
  "avid": {
    "source": "ensr:docs.data.eth?data=context-data",
    "mimeType": "application/json"
  }
}
```

This example uses the `context-data` key in the `ensr:` URI, which means:
* `context-data` - stores the actual data bytes (read via `data(node, "context-data")`)
* `context-hash` - stores the hash of that data (read via `data(node, "context-hash")`, derived by replacing `-data` with `-hash`)

### TypeScript Implementation Example

```
import { avid } from "avid.js";

async function avidVerify({ source }) {
  const { ensName, dataKey } = avid.parseSource(source);
  const bytes = await avid.resolveData(ensName, dataKey);
  const hashRecord = await avid.getHashRecord(ensName, dataKey);
  return avid.verifyHash(bytes, hashRecord);
}
```

## Backwards Compatibility

This ENSIP introduces a new verification protocol and does not affect existing ENS functionality. It introduces no breaking changes. Existing ENS names without AVID records will continue to function normally.

## Security Considerations

AVID ensures integrity, not origin. Authenticity depends on control of the ENS name. While ENS does not guarantee the trustworthiness of data, it allows full inspection, which can include ERC-3668 offchain data and onchain verifications. Data resolution occurs via ENS using the `ensr:` URI scheme, ensuring data is anchored to the ENS name.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References

* RFC 2119: Key words for use in RFCs to Indicate Requirement Levels
* RFC 8174: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
* ENSIP-24: Arbitrary Data Resolution
* ENS Record URI Scheme (ensr:)
