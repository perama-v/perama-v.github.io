---
layout: page
title:  "L1 portal"
permalink: /cairo/game/L1-portal/
toc: false
---

This solidity contract accepts L1 transactions and emits messages that the StarkNet
operator will receive and use to update state in a paired L2 contract.

The purpose is to allow asset holders (DOPE, PAPER) to register ownership on L2, and then
only have to perform L2 transactions.

This is one option, the other option is to make ownership snapshots and use Merkle proofs
to claim directly on L2.

Building upon this [contract](https://www.cairo-lang.org/docs/_static/L1L2Example.sol).

```
pragma solidity ^0.6.11;

interface IStarknetCore {
    /**
      Sends a message to an L2 contract.
    */
    function sendMessageToL2(
        uint256 to_address,
        uint256 selector,
        uint256[] calldata payload
    ) external virtual;

    /**
      Consumes a message that was sent from an L2 contract.
    */
    function consumeMessageFromL2(uint256 fromAddress, uint256[] calldata payload) external virtual;
}

/**
  Demo contract for L1 <-> L2 interaction between an L2 StarkNet contract and this L1 solidity
  contract.
*/
contract L1L2Example {
    // The StarkNet core contract.
    IStarknetCore starknetCore;

    mapping(uint256 => uint256) public userBalances;

    uint256 constant MESSAGE_WITHDRAW = 0;

    // The selector of the "deposit" l1_handler.
    uint256 constant DEPOSIT_SELECTOR =
        352040181584456735608515580760888541466059565068553383579463728554843487745;

    /**
      Initializes the contract state.
    */
    constructor(IStarknetCore starknetCore_) public {
        starknetCore = starknetCore_;
    }

    function withdraw(
        uint256 l2ContractAddress,
        uint256 user,
        uint256 amount
    ) external {
        // Construct the withdrawal message's payload.
        uint256[] memory payload = new uint256[](3);
        payload[0] = MESSAGE_WITHDRAW;
        payload[1] = user;
        payload[2] = amount;

        // Consume the message from the StarkNet core contract.
        // This will revert the (Ethereum) transaction if the message does not exist.
        starknetCore.consumeMessageFromL2(l2ContractAddress, payload);

        // Update the L1 balance.
        userBalances[user] += amount;
    }

    function deposit(
        uint256 l2ContractAddress,
        uint256 user,
        uint256 amount
    ) external {
        require(amount < 2**64, "Invalid amount.");
        require(amount <= userBalances[user], "The user's balance is not large enough.");

        // Update the L1 balance.
        userBalances[user] -= amount;

        // Construct the deposit message's payload.
        uint256[] memory payload = new uint256[](2);
        payload[0] = user;
        payload[1] = amount;

        // Send the message to the StarkNet core contract.
        starknetCore.sendMessageToL2(l2ContractAddress, DEPOSIT_SELECTOR, payload);
    }
}
```