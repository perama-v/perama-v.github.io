---
layout: page
title:  "Bitwise operations"
permalink: /cairo/examples/bitwise/
toc: false
---

The bitwise operations `AND`, `XOR` and `OR` are available using the `bitwise()` module
from the Cairo common library.

-   `bitwise_and(x, y)`, the result of bitwise AND operation on x and y.
-   `bitwise_xor(x, y)`, the result of bitwise XOR operation on x and y.
-   `bitwise_or(x, y)`, the result of bitwise OR operation on x and y.

All three operations may also be obtained with the `bitwise_operations(x, y)` function.

```
%builtins bitwise

from starkware.cairo.common.cairo_builtins import BitwiseBuiltin
from starkware.cairo.common.bitwise import (bitwise_operations,
bitwise_and, bitwise_xor, bitwise_or)


func main{bitwise_ptr: BitwiseBuiltin*}() -> ():

    # Cairo works with felts, which are integers in decimal
    # representation. Define two binary numbers, a and b,
    # using powers of 2 representation.

    # Binary: a = 1100.
    let a = 1 * 2**3 + 1 * 2**2 + 0 * 2**1 + 0 * 2**0
    assert a = 1 * 8 + 1 * 4 + 0 * 2 + 0 * 1
    assert a = 12  # Decimal representation.

    # Binary: b = 1010.
    let b = 1 * 2**3 + 0 * 2**2 + 1 * 2**1 + 0 * 2**0
    assert b = 1 * 8 + 0 * 4 + 1 * 2 + 0 * 1
    assert b = 10  # Decimal representation.

    # 1100 AND 1010 = 1000.
    let (a_and_b) = bitwise_and(a, b)
    assert a_and_b = 1 * 2**3 + 0 * 2**2 + 0 * 2**1 + 0 * 2**0

    # 1100 XOR 1010 = 0110.
    let (a_xor_b) = bitwise_xor(a, b)
    assert a_xor_b = 0 * 2**3 + 1 * 2**2 + 1 * 2**1 + 0 * 2**0

    # 1100 OR 1010 = 1110.
    let (a_or_b) = bitwise_or(a, b)
    assert a_or_b = 1 * 2**3 + 1 * 2**2 + 1 * 2**1 + 0 * 2**0

    # User defined values x and y. Returns all three operations.
    let (and, xor, or) = bitwise_operations(a, b)
    # Check that they match the result above.
    assert and = a_and_b
    assert xor = a_xor_b
    assert or = a_or_b

    # Bitwise operations must be less than 251-bit.
    # let (c) = bitwise_or(2**251, 2**3)  # Fails.
    let (c) = bitwise_or(2**250, 2**3)
    assert c = 2**250 + 2**3

    return ()
end

```
Save as `program.cairo`

### Compile and run

Note that the bitwise builtin requires the `all` layout.

```
cairo-compile program.cairo --output=compiled_program.json

cairo-run --program=compiled_program.json \
    --program_input=input.json --print_output --layout=all
```


------

## StarkNet

Cairo programs may use bitwise operations and below is a preview
of how an equivalent StarkNet contract would look once they are enabled.

```sh
%lang starknet
%builtins bitwise

from starkware.cairo.common.cairo_builtins import BitwiseBuiltin
from starkware.cairo.common.bitwise import (bitwise_operations,
bitwise_and, bitwise_xor, bitwise_or)

# Preview only.
# StarkNet does not currently allow bitwise operations.
@view
func check_bitwise{bitwise_ptr: BitwiseBuiltin*}() -> ():

    # Cairo works with felts, which are integers in decimal
    # representation. Define two binary numbers, a and b,
    # using powers of 2 representation.

    # Binary: a = 1100.
    let a = 1 * 2**3 + 1 * 2**2 + 0 * 2**1 + 0 * 2**0
    assert a = 1 * 8 + 1 * 4 + 0 * 2 + 0 * 1
    assert a = 12  # Decimal representation.

    # Binary: b = 1010.
    let b = 1 * 2**3 + 0 * 2**2 + 1 * 2**1 + 0 * 2**0
    assert b = 1 * 8 + 0 * 4 + 1 * 2 + 0 * 1
    assert b = 10  # Decimal representation.

    # 1100 AND 1010 = 1000.
    let (a_and_b) = bitwise_and(a, b)
    assert a_and_b = 1 * 2**3 + 0 * 2**2 + 0 * 2**1 + 0 * 2**0

    # 1100 XOR 1010 = 0110.
    let (a_xor_b) = bitwise_xor(a, b)
    assert a_xor_b = 0 * 2**3 + 1 * 2**2 + 1 * 2**1 + 0 * 2**0

    # 1100 OR 1010 = 1110.
    let (a_or_b) = bitwise_or(a, b)
    assert a_or_b = 1 * 2**3 + 1 * 2**2 + 1 * 2**1 + 0 * 2**0

    # User defined values x and y. Returns all three operations.
    let (and, xor, or) = bitwise_operations(a, b)
    # Check that they match the result above.
    assert and = a_and_b
    assert xor = a_xor_b
    assert or = a_or_b

    # Bitwise operations must be less than 251-bit.
    # let (c) = bitwise_or(2**251, 2**3)  # Fails.
    let (c) = bitwise_or(2**250, 2**3)
    assert c = 2**250 + 2**3

    return ()
end

```
Save as `bitwise.cairo`. Compilation will be enabled in the future.



