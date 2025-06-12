---
ensip: TBD
title: Data URL Multicodec for ENS Records
status: Draft
type: ENSRC
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>, Ghadi Mhawej (justghadi.eth) <ghadi@justalab.co>
created: 2024-06-07 (originally proposed in ENSIP-TBD-2)
---

# Abstract

This ENSIP introduces a new multicodec type, `data-url`, for ENS records. This codec allows ENS names to resolve to content embedded directly in a Data URL format. A corresponding multicodec proposal has been submitted to the multiformats/multicodec repository.

Data URLs provide a standardized way to embed small files and data directly within ENS records, eliminating the need for external hosting. This approach is particularly valuable for storing multimodal data, configuration files, schemas, and other content that benefits from being directly accessible through ENS resolution.

Because Data URLs are self-descriptive with embedded MIME types, they enable applications to interpret and handle the data appropriately without requiring prior knowledge of the content type. This makes them especially useful for AI applications, where they can store prompts, model configurations, training data, or other resources that can be dynamically accessed and processed.

# Specification

This ENSIP proposes the addition of the following multicodec:

- `data-url`: `0xf3` (pending approval)
- Temporary "Private Use Area" codec: `0x3000f3`

A pull request has been submitted to the [multiformats/multicodec repository](https://github.com/multiformats/multicodec/pull/353) for approval.

## Format

**Data URL**

```
uvarint(codec2) + byte(length(MIME)) + &lt;MIME bytes as ASCII&gt; + &lt;DATA as bytes&gt;
```

- The MIME type must not exceed 255 bytes.
- Data is stored in its raw (not base64-encoded) form.

## Example (Base64 Representation for Readability)

```
data:text/plain;base64,SGVsbG8sIFdvcmxkIQ==
```

# Security Considerations

Onchain Data URLs can improve security by eliminating dependencies on external servers. Content embedded this way is immutable and transparent, reducing risks of tampering.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
