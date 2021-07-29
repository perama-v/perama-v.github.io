---
layout: page
title:  "Receive a message"
permalink: /cairo/examples/receive_message/
toc: false
---

A StarkNet contract can receive a message from L1 by using the `@l1_handler`
function decorator. The receiving function:

- Is 'actioned' by the StarkNet sequencer (who may censor the message)
- Receives the arguments:
    - from_address (the layer 1 contract address that sent the message)
    - message elements (as type `felt` in the order they were sent).

The message originates on L1 with a call to the L1 StarkNet contract function
[`sendMessageToL2()`](https://ropsten.etherscan.io/address/0x0d761163e8bdc22fec278fea0c7a95e7b2dfa3c3#writeContract)
with arguments:

- to_address (the layer 2 contract address)
- selector
- payload

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

# Stores the sum.
@storage_var
func message_sum() -> (res : felt):
end

# Receive an L1 message. The Sequencer actions this function.
@l1_handler
func receive{range_check_ptr, storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*}(from_address : felt,
        message_index_0 : felt, message_index_1 : felt):

    # Add the two parts of the message together and save.
    tempvar sum = message_index_0 + message_index_1
    message_sum.write(sum)

    return ()
end
```
Save as `receive_message.cairo`.

### Compile

Then, to compile:
```
starknet-compile receive_message.cairo \
    --output receive_message_compiled.json \
    --abi receive_message_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract receive_message_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x0267930c360c5e77881160bcfdfae38b8a168e233efe340ddc36c97f60dc950e
Transaction ID: 73543
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=73543

Returns:
{
    "block_id": 14384,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/14384) and the
[contract](https://voyager.online/contract/0x267930c360c5e77881160bcfdfae38b8a168e233efe340ddc36c97f60dc950e#state)

### Interact

This contract is actioned by the StarkNet operator, who watches the L1 StarkNet contract
awaiting messages. The message elements are summed and stored, and another function could
be added to use this value.

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.