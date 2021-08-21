---
layout: page
title:  "Assert"
permalink: /cairo/examples/assert/
toc: false
---

An `assert` statement can be used for two purposes:

- Check that the value of two variables are the same.
- Set the value of a variable that currently has no value.

```sh
%lang starknet

struct Registry:
    member val_0 : felt
    member val_1 : felt
end

@view
func asserter(test_0 : felt, test_1 : felt) -> (val_1 : felt, val_2 : felt):
    alloc_locals
    # Define a new instance of the Registry struct.
    local newRegistry : Registry
    # Set the value of a member.
    assert newRegistry.val_0 = test_0
    # Assert that the value is something else.
    # Will fail unless 77 is selected for test_0.
    assert newRegistry.val_0 = 77

    # Set the other value. This cannot fail.
    assert newRegistry.val_1 = test_1

    # Assert that two members have the same value
    # This will fail because 77 does not equal
    assert newRegistry.val_0 = newRegistry.val_1

    return (newRegistry.val_0, newRegistry.val_1)
end
```
Save as `assert.cairo`.

### Compile

Then, to compile:
```
starknet-compile assert.cairo \
    --output assert_compiled.json \
    --abi assert_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract assert_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x05d21e543bb5f44e527ea8656024315f7499076e93053b48d955f988b4c9006c.
Transaction ID: 649197.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=649197

Returns:
{
    "block_id": 30541,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/30541) and the
[contract](https://voyager.online/contract/0x5d21e543bb5f44e527ea8656024315f7499076e93053b48d955f988b4c9006c#state)

### Interact

Then, try to call the function with values `77` and `33`. The final assert statement will fail,
showing an already assigned variable will be treated as a pass/fail condition. Passing values
'77' and '77' would pass all the assert statements.

```
starknet call \
    --network=alpha \
    --address 0x5d21e543bb5f44e527ea8656024315f7499076e93053b48d955f988b4c9006c \
    --abi assert_contract_abi.json \
    --function asserter \
    --inputs 77 33

Returns:
Error: BadRequest: HTTP error ocurred. Status: 500. Text:
{
    "code": "StarknetErrorCode.TRANSACTION_FAILED",
    "message": "Error at pc=0:6:\n
        An ASSERT_EQ instruction failed: 77 != 33\n
        Cairo traceback (most recent call last):\n
        Unknown location (pc=0:12)"
}
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.