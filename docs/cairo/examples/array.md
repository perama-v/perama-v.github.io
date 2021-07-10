---
layout: page
title:  "Array"
permalink: /cairo/examples/array/
toc: false
---

Arrays are defined as a pointer. Their values are addressed by their location
in memory, relative to the pointer.

```sh
CONTRACT

```
Save as `array.cairo`.

### Compile

Then, to compile:
```
starknet-compile array.cairo \
    --output array_compiled.json \
    --abi array_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract array_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x614d00a6c7e41df9d518f0af9484f848d95d9fe17d19b596dc4777b11c76134.
Transaction ID: 526625.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=526625

Returns:
{
    "block_id": 25169,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/2516) and the
[contract](https://voyager.online/contract/0x614d00a6c7e41df9d518f0af9484f848d95d9fe17d19b596dc4777b11c76134#state)

### Interact

Then, to interact:

```
starknet call \
    --network=alpha \
    --address 0x614d00a6c7e41df9d518f0af9484f848d95d9fe17d19b596dc4777b11c76134 \
    --abi array_contract_abi.json \
    --function read_array \
    --inputs 9

Returns:
18
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.