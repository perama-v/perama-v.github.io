---
layout: page
title:  "If-Else"
permalink: /cairo/examples/if_else/
toc: false
---

Cairo supports the condition statements `if` and `else`.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

@storage_var
func criterion() -> (val : felt):
end

@storage_var
func met_criteria_memory() -> (val : felt):
end

@view
func match_count{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (count : felt):
    let (num) = met_criteria_memory.read()
    return (num)
end

@external
func is_match{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(number : felt) -> (result : felt):
    let (condition) = criterion.read()
    if number == condition:
        let (total) = do_thing()
        return (total)
    else:
        return (0)
    end
end

@external
func store_criterion{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(number : felt):
    criterion.write(number)
    return ()
end

func do_thing{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (total : felt):
    let (mem) = met_criteria_memory.read()
    met_criteria_memory.write(mem + 1)
    return (mem + 1)
end
```
Save as `if_else.cairo`.

### Compile

Then, to compile:
```
starknet-compile if_else.cairo \
    --output if_else_compiled.json \
    --abi if_else_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract if_else_compiled.json \
    --network=alpha

Deploy transaction was sent.
Contract address: 0x04ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02.
Transaction ID: 529498.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=529498

Returns:
{
    "block_id": 25294,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/25294) and the
[contract](https://voyager.online/contract/0x4ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02#state)

### Interact

Then, set the condition.

```
starknet invoke \
    --network=alpha \
    --address 0x4ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02 \
    --abi if_else_contract_abi.json \
    --function store_criterion \
    --inputs 10

Returns:
Invoke transaction was sent.
Contract address: 0x04ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02.
Transaction ID: 529530.
```
Then call is_match a few times:
```
starknet invoke \
    --network=alpha \
    --address 0x4ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02 \
    --abi if_else_contract_abi.json \
    --function is_match \
    --inputs 10

Returns:
Invoke transaction was sent.
Contract address: 0x04ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02.
Transaction ID: 529530.
```

Then test that the contract remembers the number of matches:

```
starknet call \
    --network=alpha \
    --address 0x4ac2abb486ee741ccf26af5372d3608e5e79262680e96e2f0532300de7ded02 \
    --abi if_else_contract_abi.json \
    --function match_count

Returns:
4
```


Note the following:
- An `if` statements may optionally be followed by an `else`.
- Both `if` and `else` statements must contain a `return()` statement.
- An `end` statement must follow an `if` statement or an `if-else` pair.
- `else if` statements are not supported.

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.