---
layout: page
title:  "Cairo Common Library"
permalink: /cairo/cairo-common-library/
toc: false
---

The Cairo common library contains modules that can be imported into Cairo
code. They are imported with the following syntax, where `MODULE` is the module name,
and `COMPONENT` is the function or struct to import:

```
from starkware.cairo.common.MODULE import COMPONENT
```
For example, to use the `hash2()` function from the `hash` module:
```
from starkware.cairo.common.hash import hash2
```

Available modules and their components:

- [alloc](#alloc)
    - `alloc()`. Allocate new memory segment.
- [cairo_builtins](#cairo_builtins)
    - `BitwiseBuiltin`. The struct used for a bitwise operation.
    - `HashBuiltin`. The struct used for a hash.
    - `SignatureBuiltin`. The struct used for an ECDSA signature.
- [default_dict](#default_dict)
    - `default_dict_new()`. Create new dictionary without a hint.
    - `default_dict_finalize()`. Check the default dictionary.
- [dict](#dict)
    - `dict_new()`. Create new dictionary using a hint.
    - `dict_read()`. Read dictionary.
    - `dict_write()`. Write to dictionary.
    - `dict_update()`. Update a value in a dictionary.
    - `dict_squash()`. Remove intermediate values from the dictionary.
- [dict_access](#dict_access)
    - `DictAccess`. The struct used for a dictionary.
- [find_element](#find_element)
    - `find_element()`. Find element in an array.
    - `search_sorted_lower()`. Get first value larger than `x` in a sorted array.
    - `search_sorted()`. Get first value equal to `x` in a sorted array.
- [hash](#hash)
    - `hash2()`. Get Pedersen hash of `a` and `b`.
- [hash_chain](#hash_chain)
    - `hash_chain()`. Get hash chain of a list, last to first.
- [hash_state](#hash_state)
    - `HashState`. The struct used for a state hash of a list, from first to last.
    - `hash_init()`. Initialize a state hash.
    - `hash_update()`. Add an array to a state hash.
    - `hash_update_single()`. Add a single item to state hash.
- [keccak](#keccak)
    - `unsafe_keccak()`. Compute the Keccak hash.
- [math](#math)
    - `assert_not_zero()`. Verify `value ! = 0`.
    - `assert_not_equal()`. Verify `a! = b`.
    - `assert_nn()`. Verify `a >= 0`.
    - `assert_le()`. Verify `a <= b`.
    - `assert_lt()`. Verify `a < b`.
    - `assert_nn_le()`. Verify `0 <= a <= b`.
    - `assert_in_range()`. Verify `value` is in range `[lower, upper)`.
    - `assert_le_250_bit()`. Verify `a` and `b` are in range `[0, 2**250)`.
    - `split_felt()`. Get the unsigned integer lift of a value.
    - `assert_le_felt()`. Verify `lift(a) < b`.
    - `abs_value()`. Get the absolute value.
    - `sign()`. Get the sign of a value.
    - `unsigned_div_rem()`. Get value and remainder for integer division.
    - `signed_div_rem()`. Get value and remainder for integer division allowing negatives.
- [math_cmp](#math_cmp)
    - `is_nn()`. Check if `a >= 0`.
    - `is_le()`. Check if `a <= b`.
    - `is_in_range()`. Check if `a` is in `[lower, upper)`.
    - `is_le_felt()`. Check if `lift(a) < b`.
- [memcpy](#memcpy)
    - `memcpy()`. Copy the length of a field element.
- [merkle_multi_update](#merkle_multi_update)
    - `merkle_multi_update()`. Update multiple leaves of merkle tree.
- [merkle_update](#merkle_update)
    - `merkle_update()`. Update single leaf of merle tree.
- [pow](#pow)
    - `pow()`. Get `base ** exp`.
- [registers](#registers)
    - `get_fp_and_pc()`. Get contents of `fp` and `pc`.
    - `get_ap()`. Get content of `ap`.
    - `get_label_location()`. Get runtime address of particular label in memory.
- [serialize](#serialize)
    - `serialize_word()`. Append single value to the program output.
    - `array_rfold()`. Append an array to the program output, from right to left.
    - `serialize_array()`. Append an array to the program output, from left to right.
- [set](#set)
    - `set_add()`. Add an element to a set.
- [signature](#signature)
    - `verify_ecdsa_signature()`. verifies the prover knows signature)
- [small_merkle_tree](#small_merkle_tree)
    - `small_merkle_tree()`. updates multpile leaves in merkel tree based on a squashed dictionary)
    - `merkle_multi_update()`. verify two roots
- [squash_dict](#squash_dict)
    - `squash_dict()`. verifies dict construction sequence was valid and compresses
- [uint256](#uint256)
    - `Uint256`. The struct used for 256-bit math.
    - `uint256_add()`. 256-bit `a + b`.
    - `uint256_mul()`. 256-bit `a * b`.

## alloc

See the [alloc](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/alloc.cairo) module for more details.

Returns a newly allocated memory segment. This is useful when defining dynamically allocated
arrays. As more elements are added, more memory will be allocated.

```
    from starkware.cairo.common.alloc import alloc

    # Allocate a memory segment.
    let (new_slot : felt*) = alloc()

    # Allocate a memory segment for an array of structs.
    let (local my_array : MyStruct*) = alloc()
```

## cairo_builtins

Contains three predefined structs:

- `BitwiseBuiltin`
- `HashBuiltin`
- `SignatureBuiltin`

Cairo tracks hashes and signatures in discrete areas of memory. This is because AIR
construction is simpler when they are handled separately. As such, hashes and signatures
are tracked by the compiler in a standardised way, represented with a struct with
predefined members.

The members are:

- `BitwiseBuiltin`: `x`, `y`, `x_and_y`, `x_xor_y`, and `x_or_y`.
- `HashBuiltin`: `x`, `y` and `result`.
- `SignatureBuiltin`: `pub_key` and `message`.

Therefore `my_hash.result` would access the hash of `x` and `y`, and `my_signature.pub_key`
would access the public key of `my_signature`.

The builtin is used to standardise these components and to make type declaration simple.

- A pointer to my_bitwise would be of type `BitwiseBuiltin*`.
- A pointer to my_hash would be of type `HashBuiltin*`.
- A pointer to my_signature would be of type `SignatureBuiltin*`.

A function expecting a hash as an argument:

```
from starkware.cairo.common.cairo_builtins import HashBuiltin

# The function is expecting a hash.
func use_hash(a_hash : HashBuiltin*):
    # a_hash.x
```

See the [cairo_builtins](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/cairo_builtins.cairo)
module for more details.

## default_dict

Returns a new dictionary with all values of the key-value pairs all with the same value.
The dictionary is used by StarkNet contracts, because to immediately assign a value requires
a hint, which is not availbale in StarkNet). Operations on a default dict can be performed by
using the [dict](#dict) module.

There are two functions:

- `default_dict_new()`. Create new dictionary without a hint.
- `default_dict_finalize()`. Check the default dictionary.

See the [default_dict](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/default_dict.cairo)
module for more details. See a deployed
[dictionary]({{ site.baseurl }}{% link cairo/examples/default_dict.md %}) for an example.

### default_dict_new()

Used to create a new dictionary. Returns a pointer to a dictionary that is empty, but
will return a default value for all keys.

Accepts one explicit argument:

- `default_value`, a felt representing the value that will be returned for all
undeclared keys.

Returns:

- `res`, a pointer to the `DictAccess` struct from the [dict_access](#dict_access) module.

### default_dict_finalize()

Used to ensure that the prover has correctly set the default value of a new dictionary
properly.

Accepts one implicit argument:

- `range_check_ptr`

Accepts three explicit arguments:

- `dict_accesses_start` of type `DictAccess*`, a pointer to the first instance of a dictionary
modification.
- `dict_accesses_end` of type `DictAccess*`, a pointer to the the last instance of a dictionary
modification.

Returns:

- `dict_accesses_start` of type `DictAccess*`, a pointer to the first instance of a dictionary
modification.
- `dict_accesses_end` of type `DictAccess*`, a pointer to the the last instance of a dictionary
modification.

When a new dictionary is being made, the `dict_access_start` and `dict_access_end` may
both be set to the pointer returned from `default_dict_new()`.

## dict

See the [dict](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/dict.cairo)
module for more details.

## dict_access

See the [dict_access](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/dict_access.cairo)
module for more details.

## find_element

See the [find_element](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/find_element.cairo)
module for more details.

## hash

Contains one function `Hash2()`, which computes the Pedersen hash of two elements.
As discussed in the `HashBuiltin` module, Cairo segregates hashes and requires that they
use the struct `HashBuiltin` so that it can track all the hashes.

When calling `Hash2()`, a pointer to the hash must be provided implicitly. This pointer
is called `hash_ptr`.

```
from starkware.cairo.common.hash import hash2
from starkware.cairo.common.cairo_builtins import HashBuiltin

let hash = hash2{hash_ptr : HashBuiltin*}(3, 5)
```

See the [hash](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/hash.cairo)
module for more details.

## hash_chain

Contains one function, `hash_chain()`, which is used to find the Pedersen hash of a list of
elements, where each element is hashed after adding to the list. In other words, a hash chain,
or a hash-of-a-hash-of-a-hash... etc.

For a list, `[a, b, c, d, e]`, the hash chain would be:
```
hash2(a, hash2(b, hash2(c, hash2(d, e))))
```
Or, visually:
```
|----------| Fourth
      |---------| Third
            |--------| Second
                  |-----| First
a     b     c     d     e
```

The function uses hash2() and therefore requires the special `hash_ptr` implicit
argument. The list to be hash is passed as a pointer.

```
from starkware.cairo.common.hash import hash_chain
from starkware.cairo.common.cairo_builtins import HashBuiltin

let hash = hash_chain{hash_ptr : HashBuiltin*}(list_to_hash*)
```

See the [hash_chain](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/hash_chain.cairo)
module for more details.

## hash_state

Contains one struct and four functions:

- `HashState`
- `hash_init()`
- `hash_update()`
- `hash_update_single()`
- `hash_finalize()`

Consider how the `hash_chain` module calculated a hash from the end of a list. How would more
elements be added to that list? This module creates a hash from the start of a list such
that elements can continue be added.

`HashState` is a struct with members `current_hash` and `n_words`. For a hash involving a list
of three elements, `n_words` would be 3. `HashState` uses `Hash2()` and
therefore `HashBuiltin`, so functions will expect the implicit argument `hash_ptr`.

The process is as follows:

1. Call `hash_init()`.
2. Call `hash_update()` with a list or `hash_update_single()` with a single element.
3. Call `hash_finalize()` to get the hash.


For a list, `[a, b, c, d, e]`, the hash state would be:
```
hash2(hash2(hash2(hash2(hash2(0, a), b), c), d), e)
```
Or, visually:
```
                   |----------| Fifth
             |----------| Fourth
        |---------| Third
   |--------| Second
|-----| First
0     a     b     c     d     e
```

```
from starkware.cairo.common.hash_state import hash_init,
    hash_update_single, hash_finalize
from starkware.cairo.common.cairo_builtins import HashState,
    HashBuiltin

# Get the state pointer
let hash_state_ptr = hash_init()
# Update the state pointer
let hash_state_ptr = hash_update_single{hash_ptr : HashBuiltin*}(
    hash_state_ptr)
# Get the hash
let hash = hash_finalize{hash_state_ptr : HashState*}(hash_state_ptr)
```

See the [hash_state](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/hash_state.cairo)
module for more details.

## keccak

See the [keccak](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/keccak.cairo)
module for more details.

## math

Contains 14 functions:

- [`assert_not_zero()`](#assert_not_zero).
- [`assert_not_equal()`](#assert_not_equal).
- [`assert_nn()`](#assert_nn).
- [`assert_le()`](#assert_le).
- [`assert_lt()`](#assert_lt).
- [`assert_nn_le()`](#assert_nn_le).
- [`assert_in_range()`](#assert_in_range).
- [`assert_le_250_bit()`](#assert_le_250_bit).
- [`split_felt()`](#split_felt).
- [`assert_le_felt()`](#assert_le_felt).
- [`abs_value()`](#abs_value).
- [`sign()`](#sign).
- [`unsigned_div_rem()`](#unsigned_div_rem).
- [`signed_div_rem()`](#signed_div_rem).

See below for descriptions or the [math](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/math.cairo)
module for more details.

### assert_not_zero()

Verifies that `value != 0`. The proof will fail otherwise.

```
assert_not_zero(value)
```

### assert_not_equal()

Verifies that `a != b`. The proof will fail otherwise.

```
assert_not_zero(a, b)
```

### assert_nn()

Verifies that `a >= 0` (or more precisely `0 <= a < RANGE_CHECK_BOUND`). Informally, that `a` is
non-negative ("nn"). The proof will fail otherwise. The function requires the implicit argument
``range_check_ptr``.

```
assert_nn(a)
```

### assert_le()

Verifies that `a <= b` (or more precisely `0 <= b - a < RANGE_CHECK_BOUND`). Informally, that
`a` is less
than or equal to ("le") `b`. The proof will fail otherwise. The function requires the implicit
argument ``range_check_ptr``.

```
assert_le(a, b)
```

### assert_lt()

Verifies that `a <= b - 1` (or more precisely `0 <= b - 1 - a < RANGE_CHECK_BOUND`). Informally,
that `a` is less than ("lt") `b`. The proof will fail otherwise. The function requires the
implicit argument ``range_check_ptr``.

```
assert_lt(a, b)
```

### assert_nn_le()

Verifies that `0 <= a <= b`. Informally that `a` and `b` are non-negative ("nn") and that `a` is less than
or equal to `b`. The proof will fail otherwise. The function requires the implicit argument
``range_check_ptr``.

```
assert_nn_le(a, b)
```

### assert_in_range()

Verifies that `value` is in the range `[lower, upper)`. Informally, that `value` is both greater
than or equal to `lower` and less than `upper`. The proof will fail otherwise.
The function requires the implicit argument ``range_check_ptr``.

```
assert_in_range(value, upper, lower)
```

### assert_le_250_bit()

Verifies that a and b are in the range [0, 2**250). Informally, that both a and b are non-negative
and less that the largest number possible in a binary system with 250 bits. The proof will fail
otherwise. The function requires the implicit argument ``range_check_ptr``.

```
assert_le_250_bit(a, b)
```

### split_felt()

Splits the unsigned integer lift of a field element into the higher 128 bit and lower 128 bit and
returns both numbers. The unsigned integer lift is the unique integer in the range [0, PRIME) that
represents the field element.

For example, if ``value`` = 17 * 2^128 + 8, then ``high`` = 17 and
``low`` = 8.

The function requires the implicit argument ``range_check_ptr``.

```
let (high, low) = split_felt(value)
```

See notes on [integer lift]({{ site.baseurl }}{% link cairo/background/integer_lift.md %})
for more information.

### assert_le_felt()

Verifies that the unsigned integer lift (as a number in the range [0, PRIME)) of a is lower than or
equal to that of b. Informally, that the integer of the larger component of the field element is
less than the integer of the smaller component. The proof will fail otherwise. The function requires
the implicit argument ``range_check_ptr``.

For example, the proof for assert_le_felt(17 * 2^128 + 8) would fail because because 17>8.

```
assert_le_felt(value)
```

### abs_value()

Returns the absolute value of a value. Informally, the function returns the value provided with any
negative sign removed. The function requires the implicit argument ``range_check_ptr``.

```
abs_value(value)
```

### sign()

Returns the sign of value: -1, 0 or 1. Informally, for positive numbers the function returns 1, for
negative numbers the function returns -1 and for zero the function returns 0. The function requires
the implicit argument ``range_check_ptr``.

```
let value_sign = sign(value)
```

### unsigned_div_rem()

Returns q and r such that 0 <= q < rc_bound, 0 <= r < div and value = q * div + r. Informally, the
function returns the quotient and remainder for a value and divisor, ignoring negative values. The
function requires the implicit argument ``range_check_ptr``.

```
let (unsigned_quotient, remainder) = unsigned_div_rem(value, divisor)
```

### signed_div_rem()

Returns q and r such that 0 <= q < rc_bound, 0 <= r < div and value = q * div + r. Informally, the
function returns the quotient and remainder for a value and divisor, ignoring negative values. The
function requires the implicit argument ``range_check_ptr``.

```
let (signed_quotient, remainder) = unsigned_div_rem(value, divisor)
```

## memcpy

See the [memcpy](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/memcpy.cairo)
module for more details.

## merkle_multi_update

See the [merkle_multi_update](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/merkle_multi_update.cairo)
module for more details.

## merkle_update

See the [merkle_update](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/merkle_update.cairo)
module for more details.

## pow

See the [pow](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/pow.cairo)
module for more details.

## registers

See the [registers](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/registers.cairo)
module for more details.

## serialize

See the [serialize](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/serialize.cairo)
module for more details.

## set

See the [set](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/set.cairo)
module for more details.

## signature

See the [signature](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/signature.cairo)
module for more details.

## small_merkle_tree

See the [small_merkle_tree](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/small_merkle_tree.cairo)
module for more details.

## squash_dict

See the [squash_dict](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/squash_dict.cairo)
module for more details.

## uint256

Note that this module is not currently enabled by the StarkNet compiler and cannot
be used until that happens.

Contains one struct and two functions:

- `Uint256`
- `uint256_add()`
- `uint256_mul()`

The struct `Uint256` is used to represent an unsigned integer with 256 bits, and is handled
by the StarkNet compiler. It is a number represented as a 256 character long sequence
of 0's and 1's, divded into `high` and `low` components. These two values are the
members of the struct.

The breakdown of a number into higher and lower parts is shown below, first in
decimal, then in binary (with the middle numbers removed).
```
my_decimal = 8498972348
        high  ^  ||  ^low

my_uint8 =   10101110
          high^ || ^low

my_uint256 = 1010100110110......10100101111110
             high^          ||           ^low
```

`my_uint256.high` begins with `10101`, and `my_uint256.low` ends with `11110`.

The functions `uint256_mul()` and `uint256_add()` both require the imiplicit argument
`range_check_ptr`, which is a pointer to the builtin `range_check`. The mul and sum functions
compute the product and sum of two numbers, and return any overflow as a carry.

```
%builtins output range_check

from starkware.cairo.common.uint256 import (uint256_add, Uint256,
    uint256_mul)
from starkware.cairo.common.serialize import serialize_word

func main{output_ptr : felt*, range_check_ptr}():
    alloc_locals
    local num1 : Uint256 = Uint256(low=0,high=13)
    local num2 : Uint256= Uint256(low=0,high=7)
    let (local mul_low : Uint256, local mul_high : Uint256) = uint256_mul(num1, num2)
    serialize_word(mul_high.low) . # 91.

    return ()
end
```

See the
[uint256](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/uint256.cairo)
module for more details.

