---
layout: page
title:  "Pointer"
permalink: /cairo/examples/pointer/
toc: false
---

A pointer is a way to communicate a data structure between functions.
Rather than passing a tuple, for example, a function can pass a pointer.
The recipient function can access the values in the tuple by using that pointer.

```sh
%lang starknet

from starkware.cairo.common.registers import get_fp_and_pc

@view
func use_pointer(number : felt) -> (res : felt):
    # The variable 'tuple' is assigned to the the pointer
    # to the tuple returned by the function.
    # The variable is of type 'felt*'.
    let (tuple) = tuple_maker(number)
    # The variable 'val' is set to the value of the third element.
    let val = tuple[2]
    return (val)
end

# The function returns a pointer to a tuple.
func tuple_maker(val : felt) -> (a_tuple : felt*):
    alloc_locals
    # Declare a tuple.
    local tuple : (felt, felt, felt) = (5, 6, 2*val)
    # Local variables are based on the frame pointer, which
    # can be accessed with this library module.
    let (__fp__, _) = get_fp_and_pc()
    # '&' denotes the address, in this case, the address of the tuple.
    return (&tuple)
end

```
Save as `pointer.cairo`.

### Compile

Then, to compile:
```
starknet-compile pointer.cairo \
    --output pointer_compiled.json \
    --abi pointer_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract pointer_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x01952f14c2b56d7e09ef19c9a466de086d2ad65da30008e47a99b8ce7d447146.
Transaction ID: 555662.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=555662

Returns:
{
    "block_id": 26445,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/26445) and the
[contract](https://voyager.online/contract/0x1952f14c2b56d7e09ef19c9a466de086d2ad65da30008e47a99b8ce7d447146#state)

### Interact

Then, to interact:

```
starknet call \
    --network=alpha \
    --address 0x1952f14c2b56d7e09ef19c9a466de086d2ad65da30008e47a99b8ce7d447146 \
    --abi pointer_contract_abi.json \
    --function use_pointer \
    --inputs 43

Returns:
86
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.