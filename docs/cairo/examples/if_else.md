---
layout: page
title:  "If-Else"
permalink: /cairo/examples/if_else_variable/
toc: false
---

Cairo supports the condition statements `if` and `else`.

```sh
%lang starknet
%builtins pedersen

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.core.storage.storage import Storage

@storage_var
func criterion() -> (val : felt):
end

@view
func is_match{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*}(
        number : felt) -> (result : felt):
    let (condition) = criterion.read()
    if number == condition:
        return (1)
    else:
        return (0)
    end
end

@external
func store_criterion{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*}(
        number : felt):
    criterion.write(number)
    return()
end
```
Save as `if_else_variable.cairo`.

### Compile

Then, to compile:
```
starknet-compile if_else_variable.cairo \
    --output if_else_variable_compiled.json \
    --abi if_else_variable_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract if_else_variable_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x050fe7a489650d9851d0fae0237c604af1ddfa9f4ebbc1008f7d9f689511b8d0.
Transaction ID: 529003.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=529003

Returns:
{
    "block_id": 25272,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/25272) and the
[contract](https://voyager.online/contract/0x50fe7a489650d9851d0fae0237c604af1ddfa9f4ebbc1008f7d9f689511b8d0#state)

### Interact

Then, set the condition.

```
starknet invoke \
    --network=alpha \
    --address 0x50fe7a489650d9851d0fae0237c604af1ddfa9f4ebbc1008f7d9f689511b8d0 \
    --abi if_else_variable_contract_abi.json \
    --function store_criterion \
    --inputs 88

Returns:
Invoke transaction was sent.
Contract address: 0x050fe7a489650d9851d0fae0237c604af1ddfa9f4ebbc1008f7d9f689511b8d0.
Transaction ID: 529032.
```

Then test with a new value as a call function:

```
starknet call \
    --network=alpha \
    --address 0x50fe7a489650d9851d0fae0237c604af1ddfa9f4ebbc1008f7d9f689511b8d0 \
    --abi if_else_variable_contract_abi.json \
    --function is_match \
    --inputs 88

Returns:
1
```


Note the following:
- An `if` statements may optionally be followed by an `else`.
- Both `if` and `else` statments must contain a `return()` statement.
- An `end` statement must follow an `if` statement or an `if-else` pair.
- `else if` statements are not supported.

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.