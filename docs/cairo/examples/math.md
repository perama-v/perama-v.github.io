---
layout: page
title:  "Math"
permalink: /cairo/examples/math/
toc: false
---

The Cairo common library has modules for some common operations.

The following are available:

- Not zero:
    - `assert_not_zero(val)`. Asserts that `val` is not zero.
- Not equal:
    - `assert_not_equal(a, b)`. Asserts that `a` is not equal to `b`.
- Not negative:
    - `assert_nn(val)`. Asserts that `val` is not negative.
- Less than or equal to:
    - `assert_le(a, b)`. Asserts that `a` is less than or equal to `b`.
- Less than:
    - `assert_lt(a, b)`. Asserts that `a` is less than `b`.
- Not negative and less than or equal to:
    - `assert_nn_le()`. Asserts that `a` is both not negative and less than or equal to `b`.
- In range:
    - `assert_in_range(val, a, b)`. Asserts that `val` is both greater than or equal to `a`,
    and less than `b`.
- 250-bit:
    - `assert_250_bit(val)`. Asserts that `val` is smaller then the maximum value in 250-bit space.
- Split felt:
    - `split_felt(val)`. Returns `high` and `low` parts of `val`.
    - See [integer lifts]({{ site.baseurl }}{% link cairo/background/integer_lift.md %}).
- Less than or equal to (with split felt):
    - `assert_le_felt(a, b)`. Asserts that `a` is less than or equal to `b` using split felt
    method.
    - See [integer lifts]({{ site.baseurl }}{% link cairo/background/integer_lift.md %})
- Less than (with split felt):
    - `assert_lt_felt(a, b)`. Asserts that `a` is less than `b` using split felt
    method.
    - See [integer lifts]({{ site.baseurl }}{% link cairo/background/integer_lift.md %})
- Absolute value:
    - `abs_value(val)`. Returns `val` as a positive value.
- Sign:
    - `sign(val)`. Returns one of `-1`, `0` or `1` for a `val` that is negative, zero or
    positive respectively.
- Unsigned division remainder:
    - `unsigned_div_rem(value, div)`. Returns the quotient `q` and remainder `r` from integer
    division of `value/div` as positive values.
- Signed division remainder:
    - `signed_div_rem(value, div)`. Returns the quotient `q` and remainder `r` from integer
    division of `value/div`, with quotient sign either positive or negative.
    - Handles integer and modulo operations with negative numbers the same way Python does, where
    `-100 // 3` returns `-34` and `-100 % 3` returns `2`.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.math import (
    assert_not_zero, assert_not_equal, assert_nn, assert_le, assert_lt, assert_nn_le, assert_in_range, assert_250_bit, split_felt, assert_le_felt, assert_lt_felt, abs_value, sign, unsigned_div_rem, signed_div_rem)


# Accepts two numbers for integer division (signed, unsigned).
# Demonstrates successful cases of math operations.
@view
func check_values{range_check_ptr}(
    num_1: felt, num_2: felt) -> (
    u_quot: felt, u_rem: felt, s_quot: felt, s_rem: felt):


    alloc_locals
    assert_not_zero(100)
    assert_not_zero(-100)
    # assert_not_zero(0)  # Fails.

    assert_not_equal(100, 150)
    # assert_not_equal(100, 100)  # Fails.

    assert_nn(100)
    assert_nn(0)
    # assert_nn(0)  # Fails.

    assert_le(100, 150)
    assert_le(100, 100)
    # assert_le(100, 50)  # Fails.

    assert_lt(100, 150)
    # assert_lt(100, 100)  # Fails.
    # assert_lt(100, 150)  # Fails.

    assert_nn_le(100, 150)
    # assert_nn_le(-100, 150)  # Fails.
    # assert_nn_le(100, 50)  # Fails.

    assert_in_range(150, 100, 200)
    assert_in_range(150, -100, 200)
    # assert_in_range(50, 100, 200)  # Fails.

    assert_250_bit(9234)
    assert_250_bit(2**250 - 1)
    # assert_250_bit(2**250)  # Fails.
    # assert_250_bit(-100)  # Fails.

    let (high_1, low_1) = split_felt(100 * 2**128 + 150)
    assert high_1 = 100  # Just crosses mid-point (128 bits)
    assert low_1 = 150

    let (high_2, low_2) = split_felt(100 * 2**130 + 150)
    assert high_2 = 100 * 2 ** 2  # A bit more than high_1
    assert low_2 = 150

    assert_le_felt(150, 200)
    assert_le_felt(150, 150)
    # assert_le_felt(150, 50)  # Fails.

    assert_lt_felt(150, 200)
    # assert_lt_felt(150, 150)  Fails.
    # assert_lt_felt(150, 50)  Fails.

    let (local a) = abs_value(-150)
    assert a = 150

    let (local b) = sign(0)
    assert b = 0
    let (local c) = sign(-50)
    assert c = -1

    let (local d, e) = unsigned_div_rem(100, 3)
    assert d = 33
    assert e = 1

    let (local u_quot, u_rem) = unsigned_div_rem(num_1, num_2)

    # This check must be less than 2 ** 64
    let RANGE_CHECK_BOUND = 2 ** 20

    let (local f, g) = signed_div_rem(-100, 3, RANGE_CHECK_BOUND)
    # To have these asserts pass, the -1 and +1 have been added to
    # the expected values.
    assert f = -34
    assert g = 3

    let (local s_quot, s_rem) = signed_div_rem(-num_1, num_2,
        RANGE_CHECK_BOUND)

    return (u_quot, u_rem, s_quot, s_rem)
end
```
Save as `math.cairo`.

### Compile

Then, to compile:
```
starknet-compile math.cairo \
    --output math_compiled.json \
    --abi math_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract math_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x069c755d08c37685d6a1705277c59567fe1f764d009354d8987ea49d134dcc73
Transaction ID: 169365
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=169365

Returns:
{
    "block_id": 38624
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/36224) and the
[contract](https://voyager.online/contract/0x69c755d08c37685d6a1705277c59567fe1f764d009354d8987ea49d134dcc73#state)

### Interact

Then, to interact supply a divisor and remainder (1000, 501):

```
starknet call \
    --network=alpha \
    --address 0x69c755d08c37685d6a1705277c59567fe1f764d009354d8987ea49d134dcc73 \
    --abi math_contract_abi.json \
    --function check_values \
    --inputs 1000 501

Returns:
1 499 3618502788666131213697322783095070105623107215331596699973092056135872020479 2
```
Recall that the prime used for Cairo is:

```
>>> prime = 2**251 + 17 * 2**192 + 1`
>>> prime
3618502788666131213697322783095070105623107215331596699973092056135872020481
```
So the result (`...0479`) is two less than the largest number (`...0481`),
it is therefore qeuivalent to `-2`.

So the result for signed_div_rem(-1000, 501) is (-2, 2), which matches the Python equivalent:

```
>>> -1000 // 501
-2
>>> -1000 % 501
2
```


Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.