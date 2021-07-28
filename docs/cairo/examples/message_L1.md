---
layout: page
title:  "Message to L1"
permalink: /cairo/examples/message_L1/
toc: false
---

A starknet contract (L2) may specify a message for an Ethereum contract (L1) to receive.

This has three steps: generate, verify & digest.

### 1. A custom contract on StarkNet

- Generates the message.
- An application-specific contract such as the The `message_L1.cairo` contract deployed below.
- Features the common library module [`send_message_to_l1()`](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/starknet/common/messages.cairo).

### 2. The StarkNet contract on Ethereum

- Verifies the validity of the message.
- The `StarkNet.sol` deployed by StarkWare and [live on Ropsten](https://ropsten.etherscan.io/address/0x0d761163e8bdc22fec278fea0c7a95e7b2dfa3c3#code)
- Features the write function [`consumeMessageFromL2()`](https://ropsten.etherscan.io/address/0x0d761163e8bdc22fec278fea0c7a95e7b2dfa3c3#writeContract).
    - This computes the hash tying together the message and both the L2 and L1 contracts.
        - The L2 conract address (function argument, but could be stored in an L1 contract
        for security)
        - The calling `msg.sender` address (custom ethereum contract)
        - The message (some application logic. E.g., user and amount)
    - Checks that the hash is stored as a fact verified by the STARK validity proof.
    - Responds in the affirmative, and then removes the hash from storage.

### 3. A custom contract on Ethereum

- Digests the message.
- This example uses the the demo [`L1L2Example.sol`](https://ropsten.etherscan.io/address/0xce08635cc6477f3634551db7613cc4f36b4e49dc#code)
contract, deployed by StarkNet.
- Features the write function [`withdraw()`](https://ropsten.etherscan.io/address/0xce08635cc6477f3634551db7613cc4f36b4e49dc#writeContract)
    - This calls the `StarkNet.sol` contract using the address stored in the variable
    `starknetCore` when the contract was deployed.
    - See the line: `starknetCore.consumeMessageFromL2(l2ContractAddress, payload);`
    - This is a request for verification that the payload is a valid message from the
    specified L2 address.
- The purpose of this contract is to accept a message (E.g., user, withdrawl amount) and
and then call the StarkNet contract to check that is a valid message. The contract then
proceeds with some logic appropriate for the application (E.g., releasing ETH to an address).


In summary, an L1 contract can be loaded the hash of a StarkNet contract. It can then
call the official StarkNet contract, which has the ability to declare that a hash is
a `Fact` whose validity is assured by an unforgeable STARK validity proof.

- The cost of this L1 transaction is about
[50,000 gas](https://ropsten.etherscan.io/tx/0x12859dd33f925e6547c94215118b802bbb27fe53b6e4f24af9ae931c7d7b2a17).
- The cost of the verification and registration of the proof is about
[1,800,000 gas](https://ropsten.etherscan.io/tx/0x1478bb2bc4398e77d5554de6e1c63cfe096d3d2a1e5ec094c77ed014c77223b6)
and this cost can be amortized across all StarkNet users.

A StarkNet contract can message L1 by using the `send_message_to_L1()` function, as demonstrated
below.

## Example

Here is a contract to interact with the deployed `L1L2Example.sol` contract above.
It generates a message instructing the L1 contract to increase L1 balance.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.alloc import alloc
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage
from starkware.starknet.common.messages import send_message_to_l1

# The demo contract 'L1L2Contract.sol' StarkWare deployed to Ropsten.
const L1_CONTRACT_ADDRESS = (
    0xce08635cc6477f3634551db7613cc4f36b4e49dc)

@storage_var
func stored_felt() -> (res : felt):
end

# Generates a message that an L1 contract can use.
@external
func increase_L1_balance{
        syscall_ptr : felt*, storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*, range_check_ptr}():
    # Messages use the pointer 'syscall_ptr'.

    # Create a dynamically sized array for the message.
    let (message : felt*) = alloc()

    # The L1 contract expects the message to have 3 elements.
    assert message[0] = 0  # '0' is for a withdrawl L2 to L1.
    assert message[1] = 12345678 # User
    assert message[2] = 3  # Amount to increase L1 balance by.

    # Send the message.
    send_message_to_l1(
        to_address=L1_CONTRACT_ADDRESS,
        payload_size=3,
        payload=message)
    return ()
end
```
Save as `message_L1.cairo`.

### Compile

Then, to compile:
```
starknet-compile message_L1.cairo \
    --output message_L1_compiled.json \
    --abi message_L1_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract message_L1_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x02b01d301c8bec1bc6d4bdfb42de29fa3cbb0fa1c62949732224bd6528ce7509
Transaction ID: 67692
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=67692

Returns:
{
    "block_id": 13120,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/13120) and the
[contract](https://voyager.online/contract/0x2b01d301c8bec1bc6d4bdfb42de29fa3cbb0fa1c62949732224bd6528ce7509#state)

### Interact

Then, to interact, specify the nature of the message (withdraw is message type `0` in
the L1L2Demo.sol contract), a user and an amount to withdraw.

The [L1 contract](https://ropsten.etherscan.io/tx/0x12859dd33f925e6547c94215118b802bbb27fe53b6e4f24af9ae931c7d7b2a17)
reveals some details about the L2 contract, such as the
[L2 address](https://voyager.online/contract/0x5469ab95d459f0759ed907e6e872760fcf668467c92b66f5bcfb4b4d568de55),
and the users. Their balances can be obtained by using the
[`userBalances()`](https://ropsten.etherscan.io/address/0xce08635cc6477f3634551db7613cc4f36b4e49dc#readContract)
function on L1 and the [`get_balance()`](https://voyager.online/contract/0x5469ab95d459f0759ed907e6e872760fcf668467c92b66f5bcfb4b4d568de55#readContract)
function on L2.

- User `12345678` has balances:
    - `2933` on L2, which can be `withdrawn` to L1 by:
        1. Sending an L2 transaction using `message_to_l1()`, specifying
        the user and the amount to withdraw.
        (Demonstrated in the `invoke` transaction below)
        2. Waiting for the STARK proof to be verified and registered on Ropsten
        in the `StarkNet.sol` contract by the StarkNet sequencer.
        3. Sending an L1 `withdraw()` transaction containing the
        L2 contract address (`0x02b01d30...`), user and amount.
    - `400` on L1
        - Able to be 'Deposited' to L2 by using a mechanism not discussed here.
- User `887626622922744218685404352696443021086987437120` has balances:
    - `0` on L2.
    - `400` on L1


Create a message that signals the intent to withdraw (message type `0`) `3`
from the L2 balance of user `12345678`. Note that these values are hard coded
into this `message_L1.cairo` contract.

The L1 system does not require that the L2 contract has the authority to do so,
and in this way breaks the accounting balance with any other StarkNet contracts
interacting with this L1 contract.

```
starknet invoke \
    --network=alpha \
    --address 0x02b01d301c8bec1bc6d4bdfb42de29fa3cbb0fa1c62949732224bd6528ce7509 \
    --abi message_L1_contract_abi.json \
    --function increase_L1_balance

Returns:
Invoke transaction was sent.
Contract address: 0x02b01d301c8bec1bc6d4bdfb42de29fa3cbb0fa1c62949732224bd6528ce7509
Transaction ID: 67707
```

Once the proof is generated and the smart contract verifies the fact on chain,
the L1 contract on Ropsten can be called. The transaction would invoke
the external `withdraw()` function with arguments `l2ContractAddress`, `user`, and `amount`
as follows:

```
withdraw(
   0x02b01d301c8bec1bc6d4bdfb42de29fa3cbb0fa1c62949732224bd6528ce7509,
   12345678,
   3
)
```

That contract would call the
StarkNet L1 contract to check for the validity of the message. It would then increase
the balance of user `12345678` by `3`. It would understand this message to "be a true message
from L2" but it would not distinguish which L2 contract sent the message.

In this regard,
it is open to accepting messages from any contract, such as the one in this example.
This could be mitigated by storing the L2 contract address inside the L1 contract and
either asserting that value of the `l2ContractAddress` supplied as an argument to the L1
`withdraw()` function is correct, or by having the L1 contract use the stored value.

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.