---
title: ENS Package Manifest
description: An HTML package manifest format for ENS names, listing downloadable files.
contributors:
  - premm.eth
ensip:
  created: "2026-03-12"
  status: draft
---

# ENSIP-X: ENS Package Manifest

## Abstract

An ENS name can publish a package manifest describing downloadable files. The manifest is retrieved via the name's `contenthash` record (ENSIP-7) and is represented as a simple HTML document.

Files are represented as links. File sources may be standard URIs, `data:` URLs, or [ERC-8121](https://eips.ethereum.org/EIPS/eip-8121) hooks returning raw bytes.

The document may optionally include a machine-readable JSON manifest embedded inside a `<pre id="pkg-manifest">` element.

## Motivation

ENS names can identify software packages, tools, or services.

A client resolving a name such as `file-pkg.eth` can:

1. Resolve the name's `contenthash`
2. Fetch the package manifest
3. Retrieve files
4. Verify hashes

Using HTML keeps the format simple, readable, and compatible with existing web tooling.

ERC-7930 addresses provide a canonical identifier for publishers and hook targets.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Manifest Location

The package manifest MUST be published at the resource referenced by the ENS name's `contenthash` record.

The manifest MUST be an HTML document.

The `contenthash` record MAY reference any content system supported by ENS, including IPFS (per ENSIP-7).

### File Entries

Package files are represented as HTML elements with file metadata attributes. URI sources use anchor elements (`<a href="...">`); hook sources use a non-link element such as `<span>`.

Each file entry MUST include:

- File path
- File hash
- Hash algorithm

File entries MAY include a `type` attribute with the MIME type of the file content (e.g. `application/octet-stream`, `application/json`, `text/plain`). This helps clients interpret the file, especially when the source is a hook returning raw bytes.

Machine-readable metadata MUST be provided using attributes.

Example (URI with optional source indicator; only the path is linked):

```html
ipfs: <a
  href="ipfs://bafy..."
  type="application/octet-stream"
  data-path="bin/example"
  data-hash-algorithm="keccak256"
  data-hash="0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
>
  bin/example
</a>
<small title="0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa">
  keccak256: 0xaaaa...aaaa
</small>
```

The visible hash MAY be truncated. Clients MUST use the full hash stored in the attribute.

### URI Sources

If a file source is a URI, the file entry MUST be an anchor (`<a>`) with `href`.

Allowed URI types include:

- `ipfs:`
- `https:`
- `data:`
- Other standard URI schemes

### Hook Sources

If a file source is provided through an [ERC-8121](https://eips.ethereum.org/EIPS/eip-8121) hook, the file entry MUST use a non-link element (e.g. `<span>`) with a `data-hook` attribute. Hooks are resolution specifications, not links.

The hook MUST encode the call parameters, return type, and ERC-7930 address. The function selector is optional per [ERC-8121](https://eips.ethereum.org/EIPS/eip-8121); when included, it acts as a checksum for the function signature.

The element SHOULD include a `type` attribute with the MIME type of the file content (e.g. `application/octet-stream`, `application/json`, `text/plain`). This helps clients interpret the raw bytes returned by the hook.

File entries MAY include a source indicator before the path (e.g. `hook: ` or `ipfs: `) so readers can distinguish resolution method. For URI sources, only the path need be linked.

Example:

```html
hook: <span
  data-path="bin/example"
  type="application/octet-stream"
  data-hook="hook(0xc41a360a,'getFile(string)','getFile(\'bin/example\')','(bytes)',0x000100...)"
  data-hash-algorithm="keccak256"
  data-hash="0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
>
  bin/example
</span>
<small title="0xaaa...">keccak256: 0xaaaa...aaaa</small>
```

The hook encoding already contains the target address and return type. No additional target field is required.

### Publisher Identifier

If the manifest identifies a publisher or package owner, it SHOULD use an ERC-7930 address.

User interfaces MAY display the ENS primary name associated with that address.

### Optional JSON Manifest

The HTML document MAY include a JSON manifest for machine readability.

If present, the JSON manifest MUST appear inside:

```html
<pre id="pkg-manifest">
</pre>
```

The JSON manifest MAY include a `bin` field. If present:

- `bin` MUST be an object mapping unique command names to file paths (e.g. `{ "file-pkg": "bin/example", "helper": "bin/helper" }`).
- `default` MAY be present: a string that MUST match a key in `bin`. When no name is given, the client uses this bin for download and run. If `default` is omitted and there is exactly one entry in `bin`, clients MAY use that entry.

Example:

```html
<pre id="pkg-manifest">
{
  "spec": "ens-pkg-1",
  "name": "file-pkg",
  "version": "0.1.0",
  "publisher": "0x000100000101141234567890abcdef1234567890abcdef12345678",
  "bin": {
    "file-pkg": "bin/example",
    "helper": "bin/helper"
  },
  "default": "file-pkg"
}
</pre>
```

If the JSON manifest is embedded in the HTML, it MUST NOT also appear as a downloadable file link.

### Manifest Hash

If the JSON manifest is present, the HTML document SHOULD include its hash.

Example:

```html
<p
  data-manifest-hash-algorithm="keccak256"
  data-manifest-hash="0xcccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc"
>
  manifest hash: keccak256 0xcccc...cccc
</p>
```

### Download and Run CLI Example

Packages may include executable files. The `bin` field maps unique names to file paths; `default` specifies which bin is downloaded and run when no name is given. Client tooling can support download-and-run workflows.

Complete example manifest:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>file-pkg</title>
</head>
<body>
  <h1>file-pkg</h1>
  <p>Example package with executable binaries.</p>

  <h2>Files</h2>
  <ul>
    <li>
      ipfs: <a
        href="ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi"
        type="application/json"
        data-path="package.json"
        data-hash-algorithm="keccak256"
        data-hash="0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
      >
        package.json
      </a>
      <small title="0xbbbb...bbbb">keccak256: 0xbbbb...bbbb</small>
    </li>
    <li>
      hook: <span
        data-path="bin/example"
        type="application/octet-stream"
        data-hook="hook(0xc41a360a,'getFile(string)','getFile(\'bin/example\')','(bytes)',0x000100000101141234567890abcdef1234567890abcdef12345678)"
        data-hash-algorithm="keccak256"
        data-hash="0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
      >
        bin/example
      </span>
      <small title="0xaaaa...aaaa">keccak256: 0xaaaa...aaaa</small>
    </li>
    <li>
      ipfs: <a
        href="ipfs://bafybeihblobh2c5fnbidcp4b5tk5v7n2e4eqo5o6f7g8h9i0j1k2l3m4n5o6p7q"
        type="application/octet-stream"
        data-path="bin/helper"
        data-hash-algorithm="keccak256"
        data-hash="0xdddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd"
      >
        bin/helper
      </a>
      <small title="0xdddd...dddd">keccak256: 0xdddd...dddd</small>
    </li>
  </ul>

  <h2>Manifest</h2>
  <pre id="pkg-manifest">{
  "spec": "ens-pkg-1",
  "name": "file-pkg",
  "version": "0.1.0",
  "publisher": "0x000100000101141234567890abcdef1234567890abcdef12345678",
  "bin": {
    "file-pkg": "bin/example",
    "helper": "bin/helper"
  },
  "default": "file-pkg"
}</pre>

  <p
    data-manifest-hash-algorithm="keccak256"
    data-manifest-hash="0xcccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc"
  >
    manifest hash: keccak256 0xcccc...cccc
  </p>
</body>
</html>
```

Example pseudocode:

```
ensx file-pkg.eth
# Resolves contenthash, fetches manifest, downloads files, verifies hashes, runs default bin

ensx file-pkg.eth helper
# Name given: downloads and runs bin/helper
```

### Client Behavior

A client resolving a package SHOULD:

1. Resolve the ENS name's `contenthash`
2. Fetch the HTML manifest
3. If `<pre id="pkg-manifest">` exists, parse it
4. Otherwise parse file entries from HTML
5. Fetch files
6. Verify hashes

## Rationale

Using HTML as the manifest format keeps the specification simple and human-readable. Existing web tooling can serve, cache, and display manifests. The optional JSON manifest enables machine parsing without requiring a separate file. [ERC-8121](https://eips.ethereum.org/EIPS/eip-8121) hooks allow on-chain or hybrid resolution of file contents. ERC-7930 addresses provide chain-agnostic publisher identification.

## Backwards Compatibility

This ENSIP defines a new manifest format consumed by clients. It does not modify existing ENS resolution or contenthash behavior. Names that do not publish a package manifest are unaffected.

## Security Considerations

Clients MUST verify hashes before executing files and SHOULD require explicit user approval. External and hook-resolved sources MUST be treated as untrusted until verified.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
