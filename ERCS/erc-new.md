---
eip: <to be assigned>
title: Quantum-Secure Ethereum Keystore Format
description: Upgrade to the EIP-2335 keystore standard with quantum-resistant encryption, KDFs, and optional post-quantum key types
author: Parthasarathy Ramanujam <@ch4r10t33r>, Jihoon Song
discussions-to: https://ethereum-magicians.org/
status: Draft
type: Standards Track
category: ERC
created: 2025-06-17
requires: 2335
---

## Abstract

This ERC proposes an upgrade to the Ethereum keystore format defined in EIP-2335 to enhance resistance against quantum adversaries. It introduces stronger symmetric encryption, quantum-resistant key derivation functions, and optional support for post-quantum asymmetric key types.

## Motivation

As quantum computing progresses, certain cryptographic schemes, especially asymmetric primitives like ECDSA and BLS, may become vulnerable. While symmetric schemes like AES remain comparatively stronger, EIP-2335 relies on AES-128 and PBKDF2, which are insufficient under quantum threat models. This EIP aims to upgrade the keystore to be future-proof by enhancing confidentiality and forward secrecy using quantum-secure primitives.

## Specification

### Version

This document introduces version `5` of the keystore format.

### Key Changes from EIP-2335

- KDF must be `argon2id` or highly tuned `scrypt`
- Encryption must use `aes-256-gcm` or `xchacha20-poly1305` (AEAD)
- MAC is integrated via authenticated encryption tag
- Key size increased to 256 bits
- Support for post-quantum key types: `dilithium3`, `dilithium5`, `kyber768`, `falcon512`, `wots`
- Adds `quantum_secure: true` flag

### JSON Schema

The keystore format MUST conform to the following schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Quantum-Secure Ethereum Keystore",
  "type": "object",
  "properties": {
    "version": { "type": "integer", "enum": [5] },
    "crypto": {
      "type": "object",
      "properties": {
        "kdf": { "type": "string", "enum": ["argon2id", "scrypt"] },
        "kdfparams": {
          "type": "object",
          "oneOf": [
            {
              "properties": {
                "memory": { "type": "integer" },
                "iterations": { "type": "integer" },
                "parallelism": { "type": "integer" },
                "salt": { "type": "string", "pattern": "^[a-fA-F0-9]+$" }
              },
              "required": ["memory", "iterations", "parallelism", "salt"]
            },
            {
              "properties": {
                "n": { "type": "integer" },
                "r": { "type": "integer" },
                "p": { "type": "integer" },
                "salt": { "type": "string", "pattern": "^[a-fA-F0-9]+$" }
              },
              "required": ["n", "r", "p", "salt"]
            }
          ]
        },
        "cipher": { "type": "string", "enum": ["aes-256-gcm", "xchacha20-poly1305"] },
        "cipherparams": {
          "type": "object",
          "properties": {
            "nonce": { "type": "string", "pattern": "^[a-fA-F0-9]+$" },
            "tag": { "type": "string", "pattern": "^[a-fA-F0-9]+$" }
          },
          "required": ["nonce", "tag"]
        },
        "ciphertext": { "type": "string", "pattern": "^[a-fA-F0-9]+$" }
      },
      "required": ["kdf", "kdfparams", "cipher", "cipherparams", "ciphertext"]
    },
    "keytype": {
      "type": "string",
      "enum": ["secp256k1", "bls12-381", "dilithium3", "dilithium5", "kyber768"]
    },
    "description": { "type": "string" },
    "quantum_secure": { "type": "boolean", "const": true },
    "uuid": { "type": "string", "format": "uuid" },
    "path": { "type": "string" },
    "meta": {
      "type": "object",
      "properties": {
        "created": { "type": "string", "format": "date-time" },
        "version": { "type": "string" },
        "network": { "type": "string" }
      }
    }
  },
  "required": ["version", "crypto", "keytype", "quantum_secure", "uuid"]
}
```

### Example Keystore (Argon2 + AES-GCM)

```json
{
  "version": 5,
  "uuid": "b86f38a7-7ea7-4a23-82d9-9c6d2c5ef3f7",
  "keytype": "secp256k1",
  "quantum_secure": true,
  "crypto": {
    "kdf": "argon2id",
    "kdfparams": {
      "memory": 65536,
      "iterations": 4,
      "parallelism": 2,
      "salt": "a1b2c3..."
    },
    "cipher": "aes-256-gcm",
    "cipherparams": {
      "nonce": "3f2e4d...",
      "tag": "9fc8ac..."
    },
    "ciphertext": "4ba69e..."
  },
  "meta": {
    "created": "2025-06-17T21:00:00Z",
    "version": "v1.0.0",
    "network": "mainnet"
  }
}
```

## Rationale

### AES-256 vs AES-128

AES-256 offers 256-bit security, and Grover’s algorithm only reduces its effective strength to 128-bit—still considered quantum-resistant.

### AEAD for Integrity

AES-GCM and XChaCha20-Poly1305 provide both encryption and authentication in one operation, removing the need for a separate MAC field.

### Argon2id

Recommended for its strong memory-hard properties and resistance to both GPU and quantum attacks compared to PBKDF2.

### Post-Quantum Key Types

Though Ethereum currently uses ECDSA and BLS, this schema anticipates support for post-quantum algorithms like Dilithium and Kyber for future upgrades.

## Backward Compatibility

This format is **not backward-compatible** with EIP-2335. Clients must check the `version` field and handle version `5` separately. A migration tool may be provided to upgrade existing keystores.

## Security Considerations

- AES-256-GCM and XChaCha20-Poly1305 are widely audited and considered secure against quantum and classical attacks.
- Argon2id protects against brute-force cracking of passphrases even under quantum-assisted brute-force attempts.
- Post-quantum key types must be implemented carefully, as many are still undergoing standardization.

## Copyright

This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
