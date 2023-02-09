---
eip: 223
title: ERC-223 Token
author: Dexaran (@Dexaran) <dexaran@ethereumclassic.org>
type: Standard Track
category: ERC
status: Review
created: 2017-05-03
---

## Simple Summary

A standard interface for tokens with definition of `communication model` logic.

## Abstract

The following describes standard functions a token contract and contract working with specified token can implement. This standard introduces a communication model which allows for the implementation of [event handling](https://en.wikipedia.org/wiki/Event_(computing)) on the receiver's side.

## Motivation

1. This token introduces a communication model for contracts that can be utilized to straighten the behavior of contracts that interact with tokens as opposed to ERC-20 where a token transfer could be ignored by the receiving contract.
2. This token is more gas-efficient when depositing tokens to contracts.
3. This token allows for `_data` recording for financial transfers.

## Rationale

This standard introduces a communication model by enforcing the `transfer` to execute a handler function in the destination address. This is an important security consideration as it is required that the receiver explicitly implements the token handling function. In cases where the receiver does not implements such function the transfer MUST be reverted.

This standard sticks to the push transaction model where the transfer of assets is initiated on the senders side and handled on the receivers side. As the result, ERC223 transfers are more gas-efficient while dealing with depositing to contracts as ERC223 tokens can be deposited with just one transaction while ERC20 tokens require at least two calls (one for `approve` and the second that will invoke `transferFrom`).

- ERC-20 deposit: `approve` ~53K gas, `transferFrom` ~80K gas

- ERC-223 deposit: `transfer` and handling on the receivers side ~46K gas

This standard introduces the ability to correct user errors by allowing to handle ANY transactions on the recipient side and reject incorrect or improper transactions. This tokens utilize ONE transferring method for both types of interactions with tokens and externally owned addresses which can simplify the user experience and allow to avoid possible user mistakes.

One downside of the commonly used ERC-20 standard that ERC-223 is intended to solve is that ERC-20 implements two methods of token transferring: (1) `transfer` function and (2) `approve + transferFrom` pattern. Transfer function of ERC20 standard does not notify the receiver and therefore if any tokens are sent to a contract with the `transfer` function then the receiver will not recognize this transfer and the tokens can become stuck in the receivers address without any possibility of recovering them.

ERC223 standard is intended to simplify the interaction with contracts that are intended to work with tokens. ERC-223 utilizes "deposit" pattern similar to plain Ether depositing patterns - in case of ERC-223 deposit to the contract a user or a UI must simply send the tokens with the `transfer` function. This is one transaction as opposed to two step process of `approve + transferFrom` depositing.

This standard allows payloads to be attached to transactions using the `bytes calldata _data` parameter, which can encode a second function call in the destination address, similar to how `msg.data` does in an Ether transaction, or allow for public loggin on chain should it be necessary for financial transactions.

## Specification

Token
Contracts that works with tokens

### Methods

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

### Events

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

## Contract that is intended to receive ERC223 tokens

```js

function tokenReceived(address _from, uint _value, bytes calldata _data)

```

A function for handling token transfers, which is called from the token contract, when a token holder sends tokens. `_from` is the address of the sender of the token,` _value` is the amount of incoming tokens, and `_data` is attached data similar to` msg.data` of Ether transactions. It works by analogy with the fallback function of Ether transactions and returns nothing.

NOTE: `msg.sender` will be a token-contract inside the `tokenReceived` function. It may be important to filter which tokens are sent (by token-contract address). The token sender (the person who initiated the token transaction) will be `_from` inside the` tokenReceived` function.

IMPORTANT: This function must be named `tokenReceived` and take parameters` address`, `uint256`,` bytes` to match the [function signature](https://www.4byte.directory/signatures/?bytes4_signature=0xc0ee0b8a) `0xc0ee0b8a`.


## Security Considerations

This token utilizes the model similar to plain Ether behavior. Therefore replay issues must be taken into account.


## Reference implementation

This is highly recommended implementation of ERC 223 token: https://github.com/Dexaran/ERC223-token-standard/tree/development/token/ERC223


## History

- original issue: https://github.com/ethereum/EIPs/issues/223

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
