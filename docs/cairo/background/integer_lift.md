---
layout: page
title:  "Integer lifting"
permalink: /cairo/background/integer_lift/
toc: false
---

Cairo programs use large numbers and sometimes it is userful to be able to
perform checks on these numbers as part of some operation. One useful technique
is to split the field element.

Firstly, to split a number is to divide it in two, as in with a visual divide
between the left and right halves of the number.

## Decimal introduction

Consider the following:

Decimal number: `17,364,521`
- To split is to divide into two parts (`1736` and `4521`)
- High part `1736`, low part `4521`.
- So another way is to represent the number is: `1736 * 10 ** 4 + 4521`
- You lift the integers from that representation get the high and low parts.
- The `4` in `10 ** 4` comes from the idea of seeing how big the number is and dividing the power by
two.
    - `17,364,521` has 8 digits, so it is of order `~10 ** 8` (numbers less than `10 ** 9`).
    - `8 / 2` is `4`. So the high number has additional `10 ** 4` zeros.
    - The system is decimal (base 10), so using `10 ** (8 - 8/2)` tells you how to split the number.
    - `high * 10 ** 4 + low`
    - `high * base  **  (order - order / 2) + low`
    - See below for a base 2 (binary) example.

## Binary equivalent

If the same operation is performed in binary, the same process is followed.

Binary number: `10110011`
- Split into two parts (`1011` and `0011`).
- High part `1011`, low part `0011`.
- The order is `8`, the base is `2` (binary number system)
- High number will have additional `4` (`8/2`) zeros.
- `high * 2 ** 4 + low`.
- `1011 * 2 ** 4 + 0011`.
- High: `1011`, low: `0011`.

## Larger example

Cairo uses numbers just less than 256-bit, rather than the 8-bit example above.

So for a number in Cairo, the integer lift would take the number, represent it as a
256 binary number, then divide it into two 128 (256/2) bits.

- Original number:
    - `100101010110101....101010100101001` (256 digits long)
- Split number:
    - `100101010110101...` (first 128 digits) and
    - `...101010100101001` (last 128 digits)
- With integer lifts:
    - `high * 2 ** 128 + low`
    - `high` is a number that starts with `100101010110101...`
    - `low` is a number that ends with `...101010100101001`

## Use case

The ability to divide a number in this way allows for more efficient operations,
such as finding a neighbouring node in a binary tree.

In Cairo there are modules in the common library that take a field element, represent it as
a binary number and then split the number as per the above examples.

For example, consider the 256-bit number:

`number = 0010000000...(more zeros)...0000001`

Divided into high and low parts:

- High: `0010000000...000`
- Low:  `000...0000000001`

It is plain to see that `high` is much greater than `low`. This might be important to know
as part of some tree operation.

Another way to represent this might be two 128-bit components:
`00100.. * 2 ** 128 + ..00001`

or

`(2 ** 126) * 2 ** 128 + 1`

## Relationship to Cairo common library

So the operation:

`split_felt(number)` that returns `high` and `low`.

Would return:

- `high = 2 ** 126`
- `low = 1`

The comparison operator:

`is_le_felt(a, b)`

Looks are two numbers `a`, and `b` and compares their respective `high` lifts.
For example:

- `a = 0010000000...(more zeros)...0000001` (same as above)
    - `a_high = 0010000000...000`
- `b = 0001000000...(more zeros)...0000001`
    - `b_high = 0001000000...000`

It can be seen that `a` is a larger number than `b`.

The `is_le_felt(a, b)` function checks if `a_high` is less than `b_high`.

It would return:

`0` (false), because `a_high` is not less than `b_high`.

This method of checking the size of two numbers is more efficient (than a standard
comparison) in some situations.

## Split felt implementation in Cairo

In the implementation of the function in the Cairo `math` module, the number to be split
is parsed in the python hint as follows using bitwise shifts.

```
ids.high = ids.value >> 128
```

The high split is calculated by taking the 256-bit python integer, shifting it to the right
by 128 places, thus removing the right-hand most 128 binary digits.

```
         128 bits                128 bits        = 256 bits total
|----------------------||----------------------|
1110101010101...010101011011011...11011011011011
^Highest bit           ^^split point           ^Lowest bit
(lost significant)                              (least significant)

value >> 128, the left half moves right, the right half is lost.
                                 128 bits         = 128 bits total
         --->           |----------------------|xxxxxxxx(lost)xxxxxxx
                        1110101010101...01010101
                        ^Highest bit          ^^split point
```

The low split is calculated by:
```
ids.low = ids.value & ((1 << 128) - 1)
```
Which saves the low 128 bits by creating a new number comprising only 128 1's.
Bitwise AND (&) is then used, which results in the loss of the high 128 bits.
```
         128 bits                128 bits        = 256 bits total
|----------------------||----------------------|
1110101010101...010101011011011...11011011011011
^Highest bit           ^^split point           ^Lowest bit
(lost significant)                              (least significant)

Bitwise AND on the 128 bit 1's.
000000000000000000000000111111111111111111111111

                                 128 bits         = 128 bits total
xxxxxxxxx(lost)xxxxxxxxx|----------------------|
                        1011011...11011011011011
                       ^^split point
```

