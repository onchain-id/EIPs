---
eip: 7xxx
title: ONCHAINID: An Onchain Identity System
description: Formalizing ONCHAINID, a self-sovereign identity system on Ethereum.
author: Joachim Lebrun (@Joachim-Lebrun), Kevin Thizy (@Nakasar), Fabian Vogelsteller (@frozeman), Tony Malghem (@TonyMalghem), Luc Falempin(@lfalempin)
discussions-to: //TBD
status: Draft
type: Standards Track
category: ERC
created: 2023-xx-xx
---

## Abstract

ONCHAINID is a blockchain-based identity system that combines the functionalities of a key manager and a claim holder. The key manager holds keys to sign actions and execute instructions, while the claim holder manages claims that can be attested by third parties or self-attested. The identity contract defined by ONCHAINID `IIdentity` integrates these functionalities, providing a comprehensive solution for individuals and organizations to enforce compliance and access digital assets.

## Motivation

The motivation behind ONCHAINID is to address the inadequacies of the existing Ethereum protocol in managing complex account structures and verifying onchain claims about an identity. The current protocol lacks a standardized way for DApps and smart contracts to check the claims about an identity, and there is no formalized way to manage these claims onchain. Moreover, the protocol does not provide a comprehensive solution for managing keys associated with an account.

ONCHAINID aims to provide a self-sovereign identity system on the blockchain that allows users to create and manage their own identities. This identification solution enables compliance and identity verifications within the pseudonymous framework of public blockchain networks. It integrates the functionalities of a key manager and a claim holder, as proposed in the Key Manager and Claim Holder proposals by Fabian Vogelsteller. However, these proposals were never formalized as EIPs, remaining at the issue state. This EIP aims to formalize these proposals and integrate them into the ONCHAINID system.

## Specification

### Key Management
Keys are cryptographic public keys, or contract addresses that have permission to operate the identity or to interact with services in its behalf.

Those permissions are represented by the purposes they key is associated with. The list of purposes is an array of `uint256` and the following purposes are introduced:

- `1`: MANAGEMENT keys, which can manage the identity (considered an Owner of the identity, a MANAGEMENT key have all permissions).
- `2`: EXECUTION keys, which can call the approve method on the identity to execute transaction sent by the identity to other contracts.
- `3`: CLAIM keys, which can add, remove and update claims on the ONCHAINID.

The structure should be as follows:

- `key`: A public key owned by this identity
  - `purpose`: `uint256[]` Array of the key types, like 1 = MANAGEMENT, 2 = EXECUTION, 3 = CLAIM
  - `keyType`: The type of key used, which would be a `uint256` for different key types. e.g. 1 = ECDSA, 2 = RSA, etc.
  - `key`: `bytes32` The public key. // for non-hex and long keys, its the Keccak256 hash of the key
  
  ```solidity
  struct Key {
    uint256[] purposes;
    uint256 keyType;
    bytes32 key;
  }
  ```
#### getKey
Returns the full key data, if present in the identity.

```solidity
function getKey(bytes32 _key) constant returns(uint256[] purposes, uint256 keyType, bytes32 key);
```

#### keyHasPurpose
Returns `TRUE` if a key is present and has the given purpose. If the key is not present it returns `FALSE`.

```solidity
function keyHasPurpose(bytes32 _key, uint256 purpose) constant returns(bool exists);
```

#### getKeysByPurpose
Returns an array of public key `bytes32` held by this identity.

```solidity
function getKeysByPurpose(uint256 _purpose) constant returns(bytes32[] keys);
```

#### addKey
Adds a `_key` to the identity. The `_purpose` specifies the purpose of the key.

**MUST** only be done by keys of purpose `1`, or the identity itself. If it's the identity itself, the approval process will determine its approval.

**Triggers Event**: `KeyAdded`

```solidity
function addKey(bytes32 _key, uint256 _purpose, uint256 _keyType) returns (bool success);
```

#### removeKey
Removes `_key` from the identity.

**MUST** only be done by keys of purpose `1`, or the identity itself. If it's the identity itself, the approval process will determine its approval.

**Triggers Event**: `KeyRemoved`

```solidity
function removeKey(bytes32 _key, uint256 _purpose) returns (bool success);
```

### Identity usage

#### execute
Passes an execution instruction to the Key Manager.
**SHOULD** require approve to be called with one or more keys of purpose `1` or `2` to approve this execution.

Execute **COULD** be used as the only accessor for `addKey`, `removeKey` and `replaceKey` and `removeClaim`.

Returns `executionId`: SHOULD be sent to the `approve` function, to approve or reject this execution.

**Triggers Event**: `ExecutionRequested`

**Triggers on direct execution Event**: `Executed`

```solidity
function execute(address _to, uint256 _value, bytes _data) returns (uint256 executionId);
```

#### approve
Approves an execution or claim addition.
This **SHOULD** require the approval of key purpose `1`, if the `_to` of the execution is the identity contract itself, to successfully approve an execution.
And **COULD** require the approval of key purpose `2`, if the `_to` of the execution is another contract, to successfully approve an execution.

**Triggers Event**: `Approved`

**Triggers on successful execution Event**: `Executed`

**Triggers on successful claim addition Event**: `ClaimAdded`

```solidity
function approve(uint256 _id, bool _approve) returns (bool success);
```

### Events

#### `KeyAdded`
MUST be triggered when `addKey` was successfully called.

```solidity
event KeyAdded(bytes32 indexed key, uint256 indexed purpose, uint256 indexed keyType);
```

#### `KeyRemoved`
MUST be triggered when `removeKey` was successfully called.

```solidity
event KeyRemoved(bytes32 indexed key, uint256 indexed purpose, uint256 indexed keyType);
```

#### `ExecutionRequested`
MUST be triggered when `execute` was successfully called.

```solidity
event ExecutionRequested(uint256 indexed executionId, address indexed to, uint256 indexed value, bytes data);
```

#### `Executed`
MUST be triggered when `approve` was called and the execution was successfully approved.

```solidity
event Executed(uint256 indexed executionId, address indexed to, uint256 indexed value, bytes data);
```

#### `Approved`
MUST be triggered when `approve` was successfully called.

```solidity
event Approved(uint256 indexed executionId, bool approved);
```

#### `KeysRequiredChanged`
MUST be triggered when `changeKeysRequired` was successfully called.

```solidity
event KeysRequiredChanged(uint256 purpose, uint256 number);
```

## Rationale


## Backwards Compatibility


## Security Considerations


## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
