---
layout: page
title:  "Math comparison operators"
permalink: /cairo/examples/math_comparison/
toc: false
---

The Cairo common library has modules that make a comparison and check for a particular
condition. True results are returned as `1`, False results are returned as `0`.

The following are available:

- Not zero:
    - `is_not_zero(val)`. Checks if `val` is not zero.
- Not negative:
    - `is_nn(val)`. Checks if `val` is not negative.
- Not negative and is less than or equal to:
    - `is_nn_le(val, a)`. Checks if `val` is not negative and is less than or equal to `a`.
- Less than or equal to:
    - `is_le(val, a)`. Checks if `val` less than or equal to `a`.
- In range:
    - `is_in_range(val, a, b)`. Checks if `val` larger than or equal to `a` and smaller than or
    equal to `b`.
- Less than or equal to (for a field element)
    - `is_le_felt(a, b)`. Checks if lift `a_high` is lower than lift `b_high`.
    - Obtain the lifts using `split_felt(val)`.
    - See [integer lifts]({{ site.baseurl }}{% link cairo/background/integer_lift.md %})
    for background details.


```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.math_cmp import (
    is_not_zero, is_nn, is_le, is_nn_le, is_in_range,
    is_le_felt)

@view
func check_values{range_check_ptr}(
    number: felt) -> (
    a: felt, b: felt, c: felt, d: felt, e: felt, f: felt):

    alloc_locals
    # 1 if number not zero.
    let (local a) = is_not_zero(number)

    # 1 if (number - 101) not negative.
    # (1 if number greater than 100)
    let (local b) = is_nn(number - 100)

    # 1 if number less than or equal to 100.
    let (local c) = is_le(number, 100)

    # 1 if number not zero
    # and is less than or equal to 100.
    let (local d) = is_nn_le(number, 100)

    # 1 if number is greater than or equal to 100
    # and less than 250.
    let (local e) = is_in_range(number, 100, 250)

    # 1 if number is less than or equal to 100
    # Using integer lift to first compare HIGHs
    # and if need, then check LOWs. For:
    # number = num_HIGH*2**128 + num_LOW
    # another_number = num2_HIGH*2**128 + num2_LOW
    let another_number = 100
    let (local f) = is_le_felt(number, another_number)

    return (a, b, c, d, e, f)
end
```
Save as `math_comparison.cairo`.

### Compile

Then, to compile:
```
starknet-compile math_comparison.cairo \
    --output math_comparison_compiled.json \
    --abi math_comparison_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract math_comparison_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x076903fdeb4b0145b5aaf5bc88c80dbeed61fe48db80576be14f2ffe16311aca
Transaction ID: 163227
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=163227

Returns:
{
    "block_id": 36929,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/36929) and the
[contract](https://voyager.online/contract/0x76903fdeb4b0145b5aaf5bc88c80dbeed61fe48db80576be14f2ffe16311aca#state)

### Interact

Then, to interact:

```
starknet call \
    --network=alpha \
    --address 0x76903fdeb4b0145b5aaf5bc88c80dbeed61fe48db80576be14f2ffe16311aca \
    --abi math_comparison_contract_abi.json \
    --function check_values \
    --inputs 75

Returns:
1 0 1 1 0 1
```
The contract returned that:
- `75` is not zero.
- `100 - 75` is not negative.
- `75` is less than or equal to 100.
- `75` is both not negative and less than or equal to 100.
- `75` is not both greater than or equal to 100 and less than 250.
- `75` is less than or equal to `100`, using the integer lift method.

Try with another number:
```
starknet call \
    --network=alpha \
    --address 0x76903fdeb4b0145b5aaf5bc88c80dbeed61fe48db80576be14f2ffe16311aca \
    --abi math_comparison_contract_abi.json \
    --function check_values \
    --inputs 175

Returns:
1 1 0 0 1 0
```
The contract returned that:
- `175` is not zero.
- `100 - 175` is negative.
- `175` is not less than or equal to 100.
- `175` is not both not negative and less than or equal to 100.
- `175` is both greater than or equal to 100 and less than 250.
- `175` is not less than or equal to `100`, using the integer lift method.


### Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.