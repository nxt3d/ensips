---
title: Content Verification for ENS Contenthash Records
author: Prem Makeig (premm.eth) <premm@unruggable.com>
discussions-to: <URL>
status: Draft
created: 2025-10-30
---

# ENSIP-TBD-21: Content Verification for ENS Contenthash Records

## Abstract

This ENSIP defines a standard for verifying data resolved through ENS `contenthash` records by storing cryptographic hashes in a `content-verifier` record. The `content-verifier` key is stored in ENSIP-24's `data()` resolver profile, with fallback to ENS `text()` records (ENSIP-5) for backwards compatibility. This enables clients to verify that ENS-resolved content has not been altered, providing data integrity verification for any format including JSON metadata files.

## Motivation

ENS `contenthash` records enable decentralized content referencing, with ENSIP-25 supporting URI links and DataURLs for greater flexibility. However, unlike IPFS which provides built-in verification through content addressing, URI and DataURL formats are not self-verifying, creating security risks when applications rely on ENS-resolved data.

This ENSIP provides a lightweight solution by storing cryptographic hashes in ENSIP-24's `data()` resolver profile (with `text()` record fallback), enabling verification that ENS-resolved content matches the stored commitment without requiring full data storage onchain. The standard is format-agnostic and particularly important for ENSIP-25 URI/DataURL formats which lack uniform verification methods.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Scope

This ENSIP defines a standard for verifying data resolved through ENS `contenthash` records. It applies to any ENS name that resolves to content via its `contenthash` record, regardless of the content format (`text`, `JSON`, `binary`, etc.). This includes gateways that may append a DNS TLD necessary to resolve an ENS `contenthash` record, such as `data.example.eth.link`. This standard can be used by token standards that use the `tokenURI` for metadata, and applications that rely on ENS-resolved data and require integrity verification.

### content-verifier Record Specification

#### Primary Storage: ENSIP-24 data() Resolver Profile

This ENSIP defines a new key in ENSIP-24's `data()` resolver profile:

- **Key**: `"content-verifier"`
- **Value**: Bytes with the following structure:
  - Byte 1: Hash type identifier (e.g., `0x00` for Keccak256)
  - Bytes 2-31: Truncated hash value (up to 30 bytes, maximum total length of 31 bytes)

The `data()` resolver profile stores key-value pairs where keys are strings and values are bytes, making it the preferred storage mechanism for the `content-verifier` value.

#### Fallback Storage: ENS Text Record

For backwards compatibility during ENSIP-24 adoption, the `content-verifier` value MAY also be stored in an ENS `text()` record:

- **Key**: `"content-verifier"`
- **Value**: A hex-encoded string (with `0x` prefix) representing the same bytes structure as above

The value MUST be encoded as a hexadecimal string prefixed with `0x` when stored in the ENS `text()` record.

#### Precedence Rule

When both the `data()` record and the `text()` record are available for the same ENS name, implementations MUST use the `data()` record value. If the `data()` record and `text()` record conflict, the `data()` record value MUST be used and the `text()` record value MUST be ignored.

### Hash Types

Currently, only Keccak256 (hash type `0x00`) is defined. For Keccak256, the first 30 bytes of the hash are stored, resulting in a total value length of 31 bytes (1 byte hash type + 30 bytes hash). Future hash types MAY be specified in this ENSIP or in a future registry.

| Hash Type | Name | Description |
|-----------|------|-------------|
| `0x00` | Keccak256 | First 30 bytes of Keccak256 hash (32 bytes truncated to 30) |
| `0x01` - `0xFF` | Reserved | Reserved for future hash algorithms |

### Hash Computation

When computing the hash for storage in the `content-verifier` record:

1. Resolve the ENS name's `contenthash` record to obtain the content location
2. Fetch the complete content data from the resolved location
3. Compute the hash of the entire content data, starting from the first byte (byte `0`) through and including the last byte of the file, preserving all bytes exactly as retrieved. The data MUST be hashed in its raw binary form, byte-for-byte, without any modification, truncation, or filtering
4. Truncate the hash if necessary (for Keccak256, take the first 30 bytes)
5. Prepend the hash type byte (e.g., `0x00` for Keccak256) to create the final value
6. Store the bytes value in the ENSIP-24 `data()` resolver profile with key `"content-verifier"`
7. Optionally, for backwards compatibility, encode the resulting bytes as a hexadecimal string with `0x` prefix and store in the ENS `text()` record with key `"content-verifier"`

### Verification Process

To verify that ENS-resolved content matches the stored commitment:

1. **Resolve the ENS name's `contenthash` record**
2. **Fetch the complete content data** using the method specified in the `contenthash` record, reading from the first byte to the last byte of the file
3. **Read the `content-verifier` record**:
   - First, attempt to read the `content-verifier` key from the ENSIP-24 `data()` resolver profile
   - If the `data()` record is not available, fall back to reading the `content-verifier` ENS `text()` record
   - If both records are available and conflict, use the `data()` record value and ignore the `text()` record value
4. **Decode if necessary**: If the value was retrieved from a `text()` record, decode the hex string to obtain the stored bytes. If retrieved from the `data()` record, the value is already bytes.
5. **Extract the hash type** from byte 1 of the stored value (e.g., `0x00` for Keccak256)
6. **Compute the hash** of the fetched content data using the specified hash algorithm:
   - Hash the complete data from the first byte through the last byte
   - Truncate if necessary (for Keccak256, take the first 30 bytes)
   - Prepend the hash type byte to create the computed value
7. **Compare the computed value** with the stored `content-verifier` value byte-by-byte
8. **Verify result**: If the values match exactly, the content is verified as unaltered

## Examples

### Example: Keccak256 with JSON Data

Consider an ENS name `agent.example.eth` that resolves via `contenthash` to a JSON metadata file containing:

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "DeFi Trading Agent",
  "description": "An autonomous trading agent for DeFi operations",
  "image": "https://example.com/agentimage.png",
  "endpoints": [
    {
      "name": "A2A",
      "endpoint": "https://agent.example/.well-known/agent-card.json",
      "version": "0.3.0"
    }
  ],
  "registrations": [
    {
      "agentId": 12345,
      "agentRegistry": "eip155:1:0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7"
    }
  ],
  "supportedTrust": ["reputation", "crypto-economic"]
}
```

The verification process:

**Content Resolution:**
- Resolve `agent.example.eth` → `contenthash` record → content location (e.g., IPFS, HTTP endpoint, etc.)
- Fetch the complete file content from the resolved location

**Hash Computation:**
- Read the entire file from the first byte to the last byte, including all characters, whitespace, and newlines
- Compute Keccak256 hash of all bytes: `0xb2b415319466d217991e86f39cff067cdada1318e913a2e8460e2fc9b5a204b4`
- Take first 30 bytes: `0xb2b415319466d217991e86f39cff067cdada1318e913a2e8460e2fc9b5a`
- Prepend hash type byte (`0x00` for Keccak256): `0x00b2b415319466d217991e86f39cff067cdada1318e913a2e8460e2fc9b5a`
- Store bytes in ENSIP-24 `data()` resolver profile with key `"content-verifier"`
- Optionally, store as hex string in ENS `text()` record for backwards compatibility: `"0x00b2b415319466d217991e86f39cff067cdada1318e913a2e8460e2fc9b5a"`

**Verification:**
- Follow the steps outlined above for verification.

### Example: ENSIP-25 URI Format

Consider an ENS name `data.example.eth` that uses ENSIP-25 to store a URI in its `contenthash` record:

- **Content Resolution**: `data.example.eth` → `contenthash` (ENSIP-25 URI format) → resolves to `https://example.com/data.json`
- **Content Fetching**: Fetch the content from the resolved URI endpoint
- **Hash Computation**: Hash all bytes from the first byte to the last byte of the fetched content, truncate to 30 bytes, prepend hash type
- **Verification**: Compare stored `content-verifier` value with computed hash to verify content integrity

Since ENSIP-25 URI formats are not self-verifying (unlike IPFS content addressing), the content-verifier record provides essential integrity verification for this content.

### Example: Binary Data

This standard also applies to non-textual data. For example, consider `image.example.eth` resolving to a PNG image file:

- **Content Resolution**: `image.example.eth` → `contenthash` → fetch PNG file bytes
- **Hash Computation**: Hash all bytes from byte `0` to the last byte of the PNG file, truncate to 30 bytes, prepend hash type
- **Verification**: Compare stored `content-verifier` value with computed hash to verify image integrity

## Rationale

ENS provides a decentralized way to reference content via `contenthash` records. With ENSIP-25, `contenthash` can now store URI links and DataURLs in addition to content-addressed formats like IPFS. However, unlike content-addressed systems such as IPFS which provide built-in cryptographic verification through content addressing, URI and DataURL formats are not self-verifying. When `contenthash` points to mutable sources, HTTP endpoints, or uses ENSIP-25 URI/DataURL formats, there is no standard mechanism to verify content integrity.

By storing a cryptographic hash in ENSIP-24's `data()` resolver profile (with fallback to ENS `text()` records), this standard enables trustless verification of ENS-resolved content regardless of the underlying storage method or content format. This is particularly valuable for ENSIP-25 URI and DataURL formats, which lack a uniform method for data verification. The standard provides a standardized verification mechanism for these formats and is intentionally format-agnostic, allowing verification of `JSON`, binary images, documents, or any other data type that can be referenced through ENS `contenthash`.

## Backwards Compatibility

This ENSIP can be used for any `contenthash` resolved data and has no backwards compatibility issues.

## Security Considerations

None.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
