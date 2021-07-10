---
layout: page
title:  "If-Else"
permalink: /cairo/examples/if_else/
toc: false
---

Cairo supports the condition statements `if` and `else`.

```sh
%lang starknet
%builtins pedersen

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.core.storage.storage import Storage

@view
func is_fifty{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*}(
        number : felt) -> (result : felt):
    if number == 50:
        return (1)
    else:
        return (0)
    end
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

Returns:
Deploy transaction was sent.
Contract address: 0x064eb772a0130110252c3a5d26b48466675352b4d7e3c17fa47263bb2ebfd5a1.
Transaction ID: 507985.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=507985

Returns:
{
    "block_id": 24354,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/) and the
[contract](https://voyager.online/contract/0x64eb772a0130110252c3a5d26b48466675352b4d7e3c17fa47263bb2ebfd5a1#state)

### Interact

Then, use the function to detect if 50 was an input.

```
starknet call \
    --network=alpha \
    --address 0x64eb772a0130110252c3a5d26b48466675352b4d7e3c17fa47263bb2ebfd5a1 \
    --abi if_else_contract_abi.json \
    --function is_fifty \
    --inputs 49

Returns:
0
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