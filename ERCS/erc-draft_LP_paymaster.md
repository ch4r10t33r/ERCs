---
title: LP paymaster.
description: Standardisation of paymaster webservices to ensure they are compatible with Chain Abstraction protocols.
author: Parthasarathy Ramanujam (@ch4r10t33r), Utkir Sobirov (@sulpiride), Vignesh Aravamudhan (@balajivignesh22)
discussions-to: https://ethereum-magicians.org/t/erc-lp-paymaster
status: Draft
type: Standards Track
category: ERC
created: 2024-08-23
requires: 4337,7715

---

## Abstract

A proposal to establish a standardized API for paymaster web services, ensuring compatibility with various chain abstraction frameworks. This proposal introduces a feature enabling [ERC-4337](./eip-4337.md) wallets to interact with paymasters to secure liquidity on a specific chain (in addition to gas), after committing to repayment on a different chain.


## Motivation

In a multi-chain environment, bridging assets is often slow and results in a poor user experience. Several chain abstraction proposals, such as OneBalance and MagicSpend++, suggest that paymasters can improve this experience by fronting liquidity for users (in addition to gas sponsorship) after validating credible commitments provided by them. To enhance interoperability and streamline the process, it is essential to standardize the API calls to these paymaster web services, ensuring compatibility across various chain abstraction proposals. This proposal aims to achieve that standardization.

## Specification

<!--
  The Specification section should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (besu, erigon, ethereumjs, go-ethereum, nethermind, or others).

  It is recommended to follow RFC 2119 and RFC 8170. Do not remove the key word definitions if RFC 2119 and RFC 8170 are followed.

  TODO: Remove this comment before submitting
-->

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### `pm_getSupportedLiquidityTokens`

We introduce a new RPC endpont `pm_getSupportedLiquidityTokens` 

#### Request Specification

```tsx
type GetSupportedLLiquidityTokensRequestParams = {
  chainId: number;
};
```

###### `pm_getSupportedLiquidityTokens` Example Parameters

```json
  {
    "chainId": "0x10",
  },
```

###### `pm_getSupportedLiquidityTokens` Example Return Value

Paymaster service MUST return a list of tokens which it supports along with the available balance

```json
  {
    "chainId": "0x10",
    "expiry": "1725395781",
    "tokens" : [
      {
        "address" : "0x..",
        "symbol" : "USDC",
        "availableBalance": "0x..",
      },
      {
        "address" : "0x..",
        "symbol" : "WETH",
        "availableBalance": "0x..",
      },
    ]
  }
```



## Rationale

<!--
  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

TBD

## Backwards Compatibility

<!--

  This section is optional.

  All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

No backward compatibility issues found.

## Test Cases

<!--
  This section is optional for non-Core EIPs.

  The Test Cases section should include expected input/output pairs, but may include a succinct set of executable tests. It should not include project build files. No new requirements may be be introduced here (meaning an implementation following only the Specification section should pass all tests here.)
  If the test suite is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`. External links will not be allowed

  TODO: Remove this comment before submitting
-->

## Reference Implementation

<!--
  This section is optional.

  The Reference Implementation section should include a minimal implementation that assists in understanding or implementing this specification. It should not include project build files. The reference implementation is not a replacement for the Specification section, and the proposal should still be understandable without it.
  If the reference implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`. External links will not be allowed.

  TODO: Remove this comment before submitting
-->

## Security Considerations

<!--
  All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

Needs discussion.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
