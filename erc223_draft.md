---
eip: 223
title: Token Standard
author: Dexaran <dexaran@ethereumclassic.org>
type: Standards Track
category: ERC
status: Review
created: 2017-5-03
---

## Simple Summary

A standard interface for tokens with definition of `communication model` logic.

## Abstract

The following describes standard functions a token contract and contract working with specified token can implement. This standard introduces a communication model which allows for the implementation of [event handling](https://en.wikipedia.org/wiki/Event_(computing)) on the receiver's side.

## Motivation



Another disadvantages of ERC20 that ERC223 will solve: 
1. Lack of `transfer` handling possibility.
2. Loss of tokens.
3. Token-transactions should match Ethereum ideology of uniformity. When a user wants to transfer tokens, he should always call `transfer`. It doesn't matter if the user is depositing to a contract or sending to an externally owned account.

Those will allow contracts to handle incoming token transactions and prevent accidentally sent tokens from being accepted by contracts (and stuck at contract's balance).

For example decentralized exchange will no more need to require users to call `approve` then call `deposit` (which is internally calling `transferFrom` to withdraw approved tokens). Token transaction will automatically be handled at the exchange contract.

The most important here is a call of `tokenReceived` when performing a transaction to a contract.

## Specification

Token
Contracts that works with tokens

## Methods

NOTE: An important point is that contract developers must implement `tokenReceived` if they want their contracts to work with the specified tokens.

If the receiver does not implement the `tokenReceived` function, consider the contract is not designed to work with tokens, then the transaction must fail and no tokens will be transferred. An analogy with an Ether transaction that is failing when trying to send Ether to a contract that did not implement `function() payable`.


#### totalSupply

```js
function totalSupply() constant returns (uint256 totalSupply)
```
Get the total token supply

#### name

```js
function name() constant returns (string _name)
```
Get the name of token

#### symbol

```js
function symbol() constant returns (bytes32 _symbol)
```
Get the symbol of token

#### decimals

```js
function decimals() constant returns (uint8 _decimals)
```
Get decimals of token

#### standard

```js
function standard() constant returns (string _standard)
```
Get the standard of token contract. For some services it is important to know how to treat this particular token. If token supports ERC223 standard then it must explicitly tell that it does.

This function **MUST** return "erc223" for this token standard. If no "standard()" function is implemented in the contract then the contract must be considered to be ERC20.

#### balanceOf

```js
function balanceOf(address _owner) constant returns (uint256 balance)
```
Get the account balance of another account with address _owner


#### transfer(address, uint)

```js
function transfer(address _to, uint _value) returns (bool)
```

Needed due to backwards compatibility reasons because of ERC20 transfer function doesn't have `bytes` parameter. This function must transfer tokens and invoke the function `tokenReceived(address, uint256, bytes calldata)` in `_to`, if _to is a contract. If the `tokenReceived` function is not implemented in ` _to` (receiver contract), then the transaction must fail and the transfer of tokens should be reverted.

#### transfer(address, uint, bytes)

```js
function transfer(address _to, uint _value, bytes calldata _data) returns (bool)
```
function that is always called when someone wants to transfer tokens.
This function must transfer tokens and invoke the function `tokenReceived (address, uint256, bytes)` in `_to`, if _to is a contract. If the `tokenReceived` function is not implemented in ` _to` (receiver contract), then the transaction must fail and the transfer of tokens should not occur. 
If `_to` is an externally owned address, then the transaction must be sent without trying to execute ` tokenReceived` in `_to`.
 `_data` can be attached to this token transaction and it will stay in blockchain forever (requires more gas). `_data` can be empty.

NOTE: The recommended way to check whether the `_to` is a contract or an address is to assemble the code of ` _to`. If there is no code in `_to`, then this is an externally owned address, otherwise it's a contract.

## Events

#### Transfer

```js
event Transfer(address indexed _from, address indexed _to, uint256 _value)
```

Triggered when tokens are transferred. Compatible with ERC20 `Transfer` event.

#### TransferData

```js
event TransferData(bytes _data)
```

Triggered when tokens are transferred and logs transaction metadata. This is implemented as a separate event to keep `Transfer(address, address, uint256)` ERC20-compatible.

## Contract to work with tokens

```js
function tokenReceived(address _from, uint _value, bytes calldata _data)
```
A function for handling token transfers, which is called from the token contract, when a token holder sends tokens. `_from` is the address of the sender of the token,` _value` is the amount of incoming tokens, and `_data` is attached data similar to` msg.data` of Ether transactions. It works by analogy with the fallback function of Ether transactions and returns nothing.

NOTE: since solidity version 0.6.0+ there is a new `reveive()` function to handle plain Ether transfers - therefore the function `tokenFallback` was renamed to `tokenReceived` to keep the token behavior more intuitive and compatible with Ether behavior.

NOTE: `msg.sender` will be a token-contract inside the `tokenReceived` function. It may be important to filter which tokens are sent (by token-contract address). The token sender (the person who initiated the token transaction) will be `_from` inside the` tokenReceived` function.

IMPORTANT: This function must be named `tokenReceived` and take parameters` address`, `uint256`,` bytes` to match the [function signature](https://www.4byte.directory/signatures/?bytes4_signature=0xc0ee0b8a) `0xc0ee0b8a`.

## Recommended implementation
This is highly recommended implementation of ERC 223 token: https://github.com/Dexaran/ERC223-token-standard/tree/development/token/ERC223


## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
