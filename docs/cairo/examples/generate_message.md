---
layout: page
title:  "Generate a message"
permalink: /cairo/examples/generate_message/
toc: false
---

A StarkNet contract can message L1 by using the `send_message_to_L1()` function,
with arguments:

- to_address
- payload_size
- payload

The message can be received by calling the L1 StarkNet contract function
[`consumeMessageFromL2()`](https://ropsten.etherscan.io/address/0x0d761163e8bdc22fec278fea0c7a95e7b2dfa3c3#writeContract)
from the address specified in the message (`to_address` above) with the arguments:

- from_address (the layer 2 contract address)
- payload

If the message is not valid, the transaction will fail.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.alloc import alloc
from starkware.starknet.common.messages import send_message_to_l1

# Generates a message that an L1 contract can use.
@external
func generate{
        syscall_ptr : felt*, range_check_ptr}():
    # Messages use the pointer 'syscall_ptr'.

    # A message is sent as a pointer to a list.
    let (message : felt*) = alloc()
    assert message[0] = 444
    assert message[1] = 333

    # Send the message with the three required arguments.
    send_message_to_l1(
        to_address=0x123123,
        payload_size=2,
        payload=message)
    return ()
end
```
Save as `generate_message.cairo`.

### Compile

Then, to compile:
```
starknet-compile generate_message.cairo \
    --output generate_message_compiled.json \
    --abi generate_message_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract generate_message_compiled.json \
    --network=alpha

Returns:
Invoke transaction was sent.
Contract address: 0x058d4e4c2e24951e43c4877127ad4201f5aeee4300a2f8a7fc93d535695c2fcc
Transaction ID: 72635
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=72635

Returns:
{
    "block_id": 14198,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/14198) and the
[contract](https://voyager.online/contract/0x58d4e4c2e24951e43c4877127ad4201f5aeee4300a2f8a7fc93d535695c2fcc#state)

### Interact

Then, to interact, invoke the function, passing a list of `2` numbers (`4444` and `3333`).

```
starknet invoke \
    --network=alpha \
    --address 0x058d4e4c2e24951e43c4877127ad4201f5aeee4300a2f8a7fc93d535695c2fcc \
    --abi generate_message_contract_abi.json \
    --function generate

Returns:
Invoke transaction was sent.
Contract address: 0x058d4e4c2e24951e43c4877127ad4201f5aeee4300a2f8a7fc93d535695c2fcc
Transaction ID: 72635
```

Once the proof is generated and the smart contract verifies the fact on chain,
the L1 contract on Ropsten can be called.

This contract specified the recipient of the message to be the address `0x123123`, which
is the only Ethereum L1 address that can receive the mmessage.

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.