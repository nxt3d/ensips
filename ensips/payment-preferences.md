---
title: Payment Preferences
description: A text record for payment recipients to declare token and chain preferences.
contributors:
  - premm.eth
ensip:
  created: "2026-03-14"
  status: draft
---

# ENSIP-X: Payment Preferences

## Abstract

This ENSIP defines the `payment-preferences` text record key. Payment recipients can use it to declare which tokens or crypto currencies and chains they prefer to receive on, with structured routing data for deterministic applications and human-readable context for semantic understanding.

## Motivation

A recipient of payments wants to receive tokens or crypto currencies, but they have different preferences around different assets and chains. This covers fungible tokens (e.g. USDC), native assets (e.g. ETH, MATIC), and non-fungible tokens (e.g. NFTs). Without a standard way to declare these preferences, payers must guess or contact the recipient, and payment flows break.

Consider a non-profit summer camp that also hosts weddings and accepts donations. It may want USDC on Base for guest payments, native ETH on Ethereum mainnet for wedding rentals, and USDC on Optimism for donations. Each use case has different routing needs.

The `payment-preferences` text record provides a structured mapping of [ERC-7930](https://eips.ethereum.org/EIPS/eip-7930) token identifiers, symbols, preferred chains, and context. Applications can parse it deterministically, and LLMs can read it for semantic meaning.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### ERC-7930 Addresses

This specification uses [ERC-7930](https://eips.ethereum.org/EIPS/eip-7930) Interoperable Addresses for both chain identification and token contract addresses. The ERC-7930 format is a binary envelope with version, chain type, chain reference, and an optional address component.

- **Chain identifier**: An ERC-7930 Interoperable Address with a zero-length address component (AddressLength=0). It identifies a chain without specifying a target address. For native assets (e.g. ETH, MATIC), the `7930addrs` list contains chain identifiers.
- **Token address**: A full ERC-7930 Interoperable Address including both chain and target address. For fungible tokens and NFTs, the `7930addrs` list contains full token contract addresses (one per chain).

Both use the same ERC-7930 format; the presence or absence of the address component distinguishes them. In the JSON, both are encoded as `0x`-prefixed hex strings.

### Text Record Key

- **Key**: `payment-preferences`
- **Format**: Markdown with an optional JSON code block followed by optional body content

The key MUST be published via `text(bytes32,string)` as defined in ENSIP-5.

### Document Structure

The content MUST use the following structure:

1. **Head matter** (optional): Title, text, and optionally an H1 heading for the code block. The H1s are optional to save data.
2. **First code block** (optional): JSON array of token routing entries. May be preceded by an optional H1 (e.g. for the routing table).
3. **Body** (optional): Optionally an H1 heading (e.g. "Semantic Instructions"), followed by H2 sections, each describing a payment preference with semantic context.

A verbose document would include: a title, some introductory text, an H1 for the code block, the code block, an H1 for the semantic section, and H2 preference sections. The H1s are optional to save data.

Deterministic applications SHOULD locate the first code block in the document, parse it as JSON, and interpret it as the set of tokens the recipient can receive without loss of funds. The full payment preferences are expressed in the H2 sections. The head matter, including the optional H1, is for human and LLM readers only. LLMs and human readers MAY use the full document, including H2 sections, for semantic understanding.

### JSON Token Routing

When present, the first code block MUST contain a JSON array. Each entry in the array represents a token or crypto currency the recipient can receive without loss of funds. The JSON defines possible tokens only; it is not the full expressed payment preferences. The full preferences, including when and how each token should be used, are expressed in the H2 sections. Entries are ordered for indexing (0, 1, 2, ...). Each entry MUST have:

| Field | Type | Description |
|-------|------|-------------|
| `symbol` | string | Token symbol (e.g. `USDC`, `ETH`, or NFT collection symbol) |
| `name` | string (optional) | Token name (e.g. `USD Coin`, `Ethereum`, or NFT collection name). ERC-721 and many tokens have distinct symbol and name. |
| `7930addrs` | array of strings | List of ERC-7930 Interoperable Addresses as `0x`-prefixed hex strings. For token contracts: full addresses (chain + contract), one per chain. For native assets (e.g. ETH, MATIC): chain identifiers only (zero-length address component). |

See the ERC-7930 section above for the distinction between chain identifiers and token addresses.

### Routing

The routing feature ensures that a token is never sent to a chain the ENS name does not support. The `7930addrs` list defines the only chains and token contracts the recipient accepts. When routing a payment, clients MUST use the ENS name's address for the target chain, resolved via the name's address records (e.g. ENSIP-9, ENSIP-24, or interoperable address resolvers). The recipient's chain-specific address is authoritative; `payment-preferences` constrains which chains may be used.

When a `payment-preferences` document is in place, payments of tokens or crypto currencies not listed in the JSON MUST NOT be made without express consent of the recipient. The JSON list defines what the recipient accepts by default.

Example:

```json
[
  {
    "symbol": "USDC",
    "name": "USD Coin",
    "7930addrs": [
      "0x00010000010114a0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
      "0x000100000002210514833589fcd6edb6e08f4c7c32d4f71b54bda02913",
      "0x0001000000010a140b2c639c533813f4aa9d7837caf62653d097ff85"
    ]
  },
  {
    "symbol": "ETH",
    "name": "Ethereum",
    "7930addrs": ["0x00010000010100", "0x000100000002210500"]
  }
]
```

### Markdown Body

After the code block, the document MAY include an optional H1 heading (e.g. "Semantic Instructions"), followed by H2 sections. Each H2 is a payment preference heading. The body under each H2 describes when and why that preference applies. H1s are optional to save data. Applications MAY reference entries in the JSON array by index (0, 1, 2, ...) to associate a preference with a specific token routing. When used, index numbers SHOULD appear in the body text of each H2 section, not in the H2 headers.

### Example: Non-Profit Summer Camp

A non-profit summer camp also serves as a wedding venue and receives donations. It publishes the following as its `payment-preferences` text record:

````markdown
# Payment Preferences

We can receive the following tokens and crypto currencies without loss of funds. See each section below for our full payment preferences.

# Supported Tokens and Currencies

```json
[
  {
    "symbol": "USDC",
    "name": "USD Coin",
    "7930addrs": [
      "0x000100000002210514833589fcd6edb6e08f4c7c32d4f71b54bda02913",
      "0x0001000000010a140b2c639c533813f4aa9d7837caf62653d097ff85"
    ]
  },
  {
    "symbol": "ETH",
    "name": "Ethereum",
    "7930addrs": ["0x00010000010100", "0x000100000002210500"]
  }
]
```

# Payment Preferences by Use Case

## Guest Payments

For room reservations and camp fees, we prefer USDC (0) on Base. This keeps fees low and settlements fast.

## Wedding Rentals

For wedding venue bookings, we require ETH (1) on Ethereum mainnet. Invoices will specify the amount and chain.

## Donations

Donations can be sent as USDC (0) on Optimism or as ETH (1). For ETH donations: payments over 10k should use ETH on Ethereum mainnet; payments under 10k should use ETH on Base.
````

In this way, tokens can be routed to the right chain(s) by default, and additional nuance can clarify when each payment type applies.

## Rationale

Using Markdown with a leading JSON code block serves two audiences. Deterministic applications (wallets, payment routers) can extract the first code block, parse the JSON, and route payments without interpreting natural language. LLMs and humans can read the full document to understand context, e.g. why a summer camp wants ETH for weddings but USDC for guest payments. The ordered list in the JSON allows H2 sections to reference preferences by index; index numbers belong in the body text, not in the headers.

## Backwards Compatibility

This ENSIP defines a new text record key. Unaware clients will ignore it. Existing behavior is unaffected.

## Security Considerations

Clients MUST validate ERC-7930 addresses and chain identifiers before using them. Recipients should ensure the JSON is well-formed to avoid parsing errors in payment applications.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
