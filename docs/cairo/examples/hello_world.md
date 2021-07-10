---
layout: page
title:  "Hello, World!"
permalink: /cairo/examples/hello_world/
toc: false
---


`%lang starknet` tells the compiler that this is a contract,
rather than a Cairo program.
```sh
%lang starknet

# Alphabet substituation cipher for each letter.
# a = 01, b = 02, etc.
const hello = 10000805121215  # 08, 05, 12, 12, 15.
const world = 10002315181204  # 23, 15, 18, 12, 04.

@view
func greeting() -> (number_1 : felt, number_2 : felt):
    return (hello, world)
end
```
Save as `hello_world.cairo`.
### Compile

Make sure you have installed the Cairo language package, which includes a compiler and tools
to interact with StarkNet.
If not, you can follow the instructions [here](https://www.cairo-lang.org/docs/quickstart.html).

Then, to compile:

```
starknet-compile hello_world.cairo \
    --output hello_world_compiled.json \
    --abi hello_world_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract hello_world_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x060450c1895da5d54d9e53dc9010a8f3f94ce3f849c2492c36b5848fe21dc44c.
Transaction ID: 524855.
```
### Interact

Call the greeting function and decode the cipher!

```
starknet call \
    --network=alpha \
    --address \
    0x060450c1895da5d54d9e53dc9010a8f3f94ce3f849c2492c36b5848fe21dc44c \
    --abi hello_world_contract_abi.json \
    --function greeting

Returns:
10000805121215 10002315181204
```

## Monitor

Check the status of any transaction:

```
starknet tx_status --network=alpha --id=524855

Returns:
{
    "block_id": 25092,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/524855) and the
[contract](https://voyager.online/contract/0x60450c1895da5d54d9e53dc9010a8f3f94ce3f849c2492c36b5848fe21dc44c#state)

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.

Visit the [voyager explorer](https://voyager.online/) to see the transactions.