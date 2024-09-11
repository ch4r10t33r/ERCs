---
title: Chain Abstracted Paymaster.
description: Standardisation of paymaster webservices to ensure they are compatible with Chain Abstraction protocols.
author: Parthasarathy Ramanujam (@ch4r10t33r), <insert co-author names here>
discussions-to: <insert actual URL here>
status: Draft
type: Standards Track
category: ERC
created: 2024-08-23
requires: 4337

---

## Abstract

A proposal to establish a standardized API for paymaster web services, ensuring compatibility with various chain abstraction frameworks. This proposal introduces a feature enabling [ERC-4337](./eip-4337.md) wallets to interact with paymasters to secure liquidity on a specific chain (in addition to gas), after committing to repayment on a different chain.


## Motivation

In a multi-chain environment, bridging assets is often slow and results in a poor user experience. Several chain abstraction proposals, such as OneBalance and MagicSpend++, suggest that paymasters can improve this experience by fronting liquidity for users (in addition to gas sponsorship) after validating credible commitments provided to them. To enhance interoperability and streamline the process, it is essential to standardize the API calls to these paymaster web services, ensuring compatibility across various chain abstraction proposals. This proposal aims to achieve that standardization.

## Specification

We introduce two new RPC endpoints `pm_getLiquidityAssets` and `pm_getSponsorWithLiquidityStub` to be implemented by paymaster web services. 

### `pm_getLiquidityAssets`

This method MUST return a list of liquidity tokens the paymaster supports along with their `tokenAddress`, `symbol` and `availableLiquidity`.

#### Request Specification

```tsx
type GetSupportedLiquidityAssetsRequestParams = {
  chainId: number;
};
```

###### `pm_getLiquidityAssets` Example Parameters

```json
  {
    "chainId": "0x10",
  },
```

###### `pm_getLiquidityAssets` Example Return Value

Paymaster service MUST return a list of tokens which it supports along with the available liquidity

```json
  {
    "chainId": "0x10",
    "expiry": "1725395781",
    "tokens" : [
      {
        "address" : "0x..",
        "symbol" : "USDC",
        "availableLiquidity": "0x..",
      },
      {
        "address" : "0x..",
        "symbol" : "WETH",
        "availableLiquidity": "0x..",
      },
    ]
  }
```

### `pm_getSponsorWithLiquidityStub`

We introduce a new RPC endpont `pm_getSponsorWithLiquidityStub` 

#### Request Specification

```tsx
// [liqudity, userOp, entryPoint, chainId, context]
type GetSponsorWithLiquidityStubParams = [
  // Information of the asset for which liquidity is expected
  {
    chainId: number;
    tokenAddress: `0x${string}`;
    // proof to resource lock/escrow.
    resourceLock: `0x${string}`;
  },
  // Below is specific to Entrypoint v0.6 but this API can be used with other entrypoint versions too
  {
    sender: `0x${string}`;
    nonce: `0x${string}`;
    initCode: `0x${string}`;
    callData: `0x${string}`;
    callGasLimit: `0x${string}`;
    verificationGasLimit: `0x${string}`;
    preVerificationGas: `0x${string}`;
    maxFeePerGas: `0x${string}`;
    maxPriorityFeePerGas: `0x${string}`;
  },
  `0x${string}`, // EntryPoint
  `0x${string}`, // Chain ID
  Record<string, any> // Context
];

type GetSponsorWithLiquidityResult = {
  paymaster?: string // Paymaster address (entrypoint v0.7)
  paymasterData?: string; // Paymaster data (entrypoint v0.7)
  paymasterAndData?: string; // Paymaster and data (entrypoint v0.6)
};
```

###### `pm_getSponsorWithLiquidityStub` Example Parameters

```json
[
  {
    "chainId": "0x10",
    "tokenAddress": "0x...",
    "resourceLock": "0x...",
  },
  {
    "sender": "0x...",
    "nonce": "0x...",
    "initCode": "0x...",
    "callData": "0x...",
    "callGasLimit": "0x...",
    "verificationGasLimit": "0x...",
    "preVerificationGas": "0x...",
    "maxFeePerGas": "0x...",
    "maxPriorityFeePerGas": "0x...",
  },
  "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789",
  "0x2105",
  {
    // This is illustrative, these are usually defined by the service provider.
    "validFrom": "0x...",
    "validUntil": "0x...",
    "policyId" : "0bfb4251-3960-4c37-87c9-d65bfcaeca13",
  }
]
```

###### `pm_getSponsorWithLiquidityStub` Example Return Value

Paymaster services MUST detect which entrypoint version the account is using and verify the validity of `resourceLock` and return the correct fields. 

For example, if using entrypoint v0.6:

```json
{
  "paymasterAndData": "0x...",
  "verificationGasLimit": "0x...",
  "preVerificationGas": "0x...",
  "callGasLimit": "0x...",  
}
```

If using entrypoint v0.7:

```json
{
  "paymaster": "0x...",
  "paymasterData": "0x...",
  "preVerificationGas": "0x...",
  "verificationGasLimit": "0x...",
  "callGasLimit": "0x...",
  "paymasterVerificationGasLimit": "0x...",
  "paymasterPostOpGasLimit": "0x1"
}
```


## Rationale

The current use of paymaster services to sponsor gas for user operations via the `pm_sponsorUserOperation` method is significantly limited. This ERC proposes extending paymaster services to also front liquidity alongside gas sponsorship. Paymasters will be required to validate the user's specified resource lock before providing the necessary paymaster data, including relevant gas fields. Since fronting liquidity may introduce overhead, the estimated gas values might be inaccurate. Therefore, `pm_getSponsorWithLiquidityStub` MAY return paymaster-specific gas limits that account for any additional gas overheads related to liquidity fronting and post-operation costs.

// TODO - Insert sequence diagram HERE

## Backwards Compatibility

Paymasters which do not support `pm_getLiquidityAssets` or `pm_getSponsorWithLiquidityStub` SHOULD return an error message if the RPC method is called.

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
