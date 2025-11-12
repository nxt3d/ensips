---
title: Agentic Verifiable and Inspectable Data (AVID)
description: A simple verification protocol for agent workflows using ENS text or data records
contributors:
  - premm.eth
ensip:
  created: "2025-01-XX"
  status: draft
---

# ENSIP-TBD: Agentic Verifiable and Inspectable Data (AVID)

## Abstract

This ENSIP defines AVID (Agentic Verifiable and Inspectable Data), a simple verification protocol for agent workflows. Any ENS record (ENSIP-5 text records or ENSIP-24 data records) can have its integrity verified by checking a corresponding hash record. The hash record is created by inserting `-hash` before any colon in the record key, or appending `-hash` if no colon exists (e.g., `agent-context` → `agent-context-hash`, `key-with-param:1` → `key-with-param-hash:1`). A client verifies byte-for-byte integrity by comparing a locally computed hash to the hash stored in the `-hash` record. The hash record MUST use the same record type as the original record. 

## Motivation

There is a need for a lightweight, deterministic way to verify data integrity in agent workflows. Current solutions often require complex signature schemes or external attestation services. IPFS-based solutions require using the IPFS network, which may not be desired. AVID provides a minimal protocol that:

- Makes arbitrary data verifiable by agents with minimal ceremony
- Binds data integrity to an ENS name that is easy to resolve across chains
- Defines a precise hashing rule so results are deterministic

AVID does not define a new transport it use ENS and does not solve authenticity by itself (authenticity comes from control of the ENS name).

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Record Types

AVID supports both ENSIP-5 text records (`text(node, key)`) and ENSIP-24 data records (`data(node, key)`). Throughout this specification, `text()` is used in examples for clarity, but `data()` works identically. The hash record MUST use the same record type as the original record (text records use `text()` for hashes, data records use `data()` for hashes).

### Terminology

**Blob.** The exact bytes of a resource as fetched by the client.

**Source.** An ENS name that anchors the authoritative hash. Data MUST be resolved via ENS records (ENSIP-5 text records or ENSIP-24 data records).

**Record key.** Any valid ENS record key (e.g., `agent-context`, `url`, `description`).

**Record type.** Either `text` (ENSIP-5) or `data` (ENSIP-24). The hash record MUST use the same record type as the original record.

**Hash record key.** The record key with `-hash` inserted before any parameter separator (`:`). If the record key contains a colon and parameter (e.g., `key-with-param:1`), the hash key becomes `key-with-param-hash:1`. If the record key has no parameter (e.g., `agent-context`), the hash key becomes `agent-context-hash`. Examples: `agent-context` → `agent-context-hash`, `key-with-param:1` → `key-with-param-hash:1`, `url` → `url-hash`.

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

The hash and data MUST be stored under the ENS name referenced by the source. Any ENS record can be verified by checking a corresponding hash record. The hash record key is created by inserting `-hash` before any parameter separator (`:`) in the record key, and the hash record MUST use the same record type as the original record.

* `{key}`: the record containing the actual data (e.g., `agent-context`, `url`, `key-with-param:1`) - accessed via `text(node, "{key}")` or `data(node, "{key}")`
* `{key-hash}`: the hash record containing the prefixed hash format (method byte + length byte + hash bytes) - accessed via `text(node, "{key-hash}")` or `data(node, "{key-hash}")`

Clients can check if a `-hash` record exists for any record to determine if AVID verification is available. Alternatively, clients may know from protocol specifications which records should have corresponding `-hash` records.

Examples:
* `agent-context` → `agent-context-hash` (e.g., `text(node, "agent-context")` → `text(node, "agent-context-hash")`)
* `key-with-param:1` → `key-with-param-hash:1` (e.g., `text(node, "key-with-param:1")` → `text(node, "key-with-param-hash:1")`)

### AVID TAGS

AVID MAY be embedded as an HTML/XML tag:

```
<AVID SRC="ensr:example.eth?text=agent-context" MIMETYPE="text/plain; charset=utf-8" />
```

The `SRC` attribute MUST contain an `ensr:` URI with either a `?text=` parameter (for text records) or a `?data=` parameter (for data records) specifying the record key. The `MIMETYPE` attribute is OPTIONAL and specifies the MIME type of the data (default: `text/plain; charset=utf-8`).

Clients MUST resolve data using the `ensr:` URI from `SRC`: fetch the record bytes from ENS via `text(node, "{key}")` or `data(node, "{key}")` where `{key}` is the record key, extract bytes exactly as received, derive the hash key by inserting `-hash` before any colon in the record key (or appending `-hash` if no colon is present), read the hash record using the same record type to determine the algorithm, compute the digest using that algorithm, and compare.

### AVID in STRUCTURED DATA

AVID metadata MAY be represented as a JSON object:

```
{
  "type": "static-resource",
  "avid": {
    "source": "ensr:example.eth?text=agent-context",
    "mimeType": "text/plain; charset=utf-8"
  }
}
```

The `avid` field MUST be an object containing:
* `source` (REQUIRED): An `ensr:` URI with either a `?text=` parameter (for text records) or a `?data=` parameter (for data records) specifying the record key
* `mimeType` (OPTIONAL): The MIME type of the data (default: `text/plain; charset=utf-8`)

Verification steps are identical: extract the `ensr:` URI from the `avid.source` field, determine the record type from the URI parameter, resolve data via that URI, derive the hash key by inserting `-hash` before any colon (or appending if no colon), read the hash record using the same record type, determine the algorithm, compute the digest, and compare.

### ENSR URI Scheme

The `ensr:` URI scheme as defined in the ENS Record URI Scheme specification is used to reference ENS records. When using `ensr:` URIs with AVID, either the `?text=` parameter (for text records) or the `?data=` parameter (for data records) MUST specify the record key. The hash key is derived by inserting `-hash` before any colon (`:`) in the record key (or appending `-hash` if no colon is present), and the hash record MUST use the same record type as the original record.

Examples:
* `ensr:example.eth?text=agent-context` → resolves to `text(node, "agent-context")`, hash at `text(node, "agent-context-hash")`
* `ensr:example.eth?text=key-with-param:1` → resolves to `text(node, "key-with-param:1")`, hash at `text(node, "key-with-param-hash:1")`

### Verification Flow

Clients MUST follow these steps for verification:

1. Parse the `ensr:` URI to extract the ENS name, record key, and record type (`?text=` → text record, `?data=` → data record). Derive the hash key by inserting `-hash` before any colon in the record key, or appending `-hash` if no colon exists.
2. Resolve the ENS name and optionally check if the hash record exists. If it doesn't exist, verification cannot proceed.
3. Fetch the record data (`text(node, "{key}")`) and the hash record (`text(node, "{key-hash}")`) using the same record type.
4. Parse the hash record to extract the method byte, length byte, and hash bytes. Determine the algorithm from the method byte (e.g., `0x00` = keccak256). If unsupported or length > 29, reject.
5. Compute the digest of the exact bytes using the determined algorithm (Hash Format) and compare with the hash bytes from the hash record. Match = success, mismatch = fail.

## Rationale

AVID provides a minimal, deterministic verification protocol that leverages existing ENS infrastructure. By inserting `-hash` before any colon in record keys (or appending if no colon exists), any ENS text record (ENSIP-5) or data record (ENSIP-24) can be verified. Hash records MUST use the same record type as the original record, ensuring consistency. The protocol focuses solely on integrity verification, leaving authenticity to ENS name control.

## Examples

### HTML TAG

```
<AVID SRC="ensr:example.eth?text=agent-context" MIMETYPE="application/json" />
```

### JSON Metadata

```
{
  "post": {
    "title": "Hello",
    "body": "..."
  },
  "avid": {
    "source": "ensr:example.eth?text=agent-context",
    "mimeType": "application/json"
  }
}
```

This example uses the `agent-context` record key:
* `agent-context` - stores the actual record data (read via `text(node, "agent-context")`)
* `agent-context-hash` - stores the hash of that data (read via `text(node, "agent-context-hash")`, matching the original record type)

### TypeScript Implementation Example

```
import { avid } from "avid.js";

async function avidVerify({ source }) {
  const { ensName, recordKey, recordType } = avid.parseSource(source);
  const hashKey = `${recordKey}-hash`;
  // Optionally check if hash record exists
  const hashExists = await avid.checkHashRecord(ensName, hashKey, recordType);
  if (!hashExists) {
    // Handle case where hash record doesn't exist
    return false;
  }
  const bytes = recordType === 'text' 
    ? await avid.resolveTextRecord(ensName, recordKey)
    : await avid.resolveDataRecord(ensName, recordKey);
  const hashRecord = await avid.getHashRecord(ensName, hashKey, recordType);
  return avid.verifyHash(bytes, hashRecord);
}
```

## Backwards Compatibility

This ENSIP introduces a new verification protocol and does not affect existing ENS functionality. It introduces no breaking changes. Existing ENS names without AVID records will continue to function normally.

## Security Considerations

AVID ensures integrity, not origin. Authenticity depends on control of the ENS name. While ENS does not guarantee the trustworthiness of data, it allows full inspection, which can include ERC-3668 offchain data and onchain verifications. Data resolution occurs via ENS records using the `ensr:` URI scheme, ensuring data is anchored to the ENS name.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References

* RFC 2119: Key words for use in RFCs to Indicate Requirement Levels
* RFC 8174: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
* ENSIP-5: Text Records
* ENSIP-24: Arbitrary Data Resolution
* ENS Record URI Scheme (ensr:)
