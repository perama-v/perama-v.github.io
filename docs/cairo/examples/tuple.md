---
layout: page
title:  "Tuple"
permalink: /cairo/examples/tuple/
toc: false
---


A tuple may contain different variable types. Elements are accessed using indices.
Where a tuple is passed to a function, the address of the tuple is sent using `&tuple`.


```sh
%lang starknet

from starkware.cairo.common.alloc import alloc
from starkware.cairo.common.registers import get_fp_and_pc

@view
func read_tuple() -> (value : felt):
    alloc_locals
    # A tuple is defined.
    local felt_tuple : (felt, felt, felt, felt) = (9, 8, 7, 18)

    # Access the tuple at the selected index.
    # Cannot pass a variable index (e.g as function input).
    let val = felt_tuple[3]
    return (val)
end

@external
func pass_tuple(index_1 : felt, index_2 : felt) -> (sum : felt):
    # A tuple is passed by sending the address of the tuple.
    alloc_locals

    # A tuple is defined.
    local the_tuple : (felt, felt, felt, felt) = (4, 6, 8, 13)

    # Get the value of the fp register.
    let (__fp__, _) = get_fp_and_pc()
    let (total) = get_sum(&the_tuple, index_1, index_2)
    return (total)
end

func get_sum(tuple_pointer : felt*, idx_1 : felt, idx_2 : felt) -> (
    total : felt):

    let sum = tuple_pointer[idx_1] + tuple_pointer[idx_2]
    return (sum)
end
```
Save as `tuple.cairo`.

### Compile

Then, to compile:
```
starknet-compile tuple.cairo \
    --output tuple_compiled.json \
    --abi tuple_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract tuple_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x03ccdbcc56abccda2a9ba7a9620f5ed7873f79674e50a381f1752180d9118cb7.
Transaction ID: 537139.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=537139

Returns:
{
    "block_id": 25632,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/25632) and the
[contract](https://voyager.online/contract/0x3ccdbcc56abccda2a9ba7a9620f5ed7873f79674e50a381f1752180d9118cb7#state)

### Interact

Then, to interact:


```
starknet invoke \
    --network=alpha \
    --address 0x3ccdbcc56abccda2a9ba7a9620f5ed7873f79674e50a381f1752180d9118cb7 \
    --abi tuple_contract_abi.json \
    --function pass_tuple \
    --inputs 2 3

Returns:
Invoke transaction was sent.
Contract address: 0x03ccdbcc56abccda2a9ba7a9620f5ed7873f79674e50a381f1752180d9118cb7.
Transaction ID: 537162.
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.