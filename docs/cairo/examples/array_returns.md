---
layout: page
title:  "array_returns"
permalink: /cairo/examples/array_returns/
toc: false
---

An external function can return an array.

In this example, an array is passed to a contract that returns
that array twice, with a struct in between.

Note that just like when passing an array to a contract the length
must be explicitly passed, the same applies for return values.

Rules:

1. Array length returned before array pointer.
2. Array length name must be the array name plus `_len`.

```sh
%lang starknet
%builtins range_check

from starkware.cairo.common.math import assert_in_range

struct ArrayCalculation:
    member a : felt
    member b : felt
    member c : felt
    member d : felt
end

@view
func duplicate_array{
        range_check_ptr
    }(
        input_arr_len : felt,
        input_arr : felt*
    ) -> (
        out_arr_1_len : felt,
        out_arr_1 : felt*,
        array_calculation : ArrayCalculation,
        out_arr_2_len : felt,
        out_arr_2 : felt*
    ):
    # Returns the two arrays with a struct between them.
    # Require the input array length to be [5, 11)
    assert_in_range(input_arr_len, 5, 11)
    # Do the calcuations.
    let one = input_arr[0] + input_arr[1]
    let two = one + input_arr[2]
    let three = two + input_arr[3]
    let four = three + input_arr[4]
    # Define the struct.
    let array_calculation = ArrayCalculation(
        a=one, b=two, c=three, d=four)
    # Return the array-struct-array sequence.
    return (
        input_arr_len,
        input_arr,
        array_calculation,
        input_arr_len,
        input_arr)
end
```
Save as `array_returns.cairo`.

### Compile

Compile all contracts
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/array_returns.cairo
```

### Test

Make a new file called `array_returns_test.py` and populate it:

```py
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet

# Enables modules.
@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

# Reusable to save testing time.
@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/array_returns.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory
    input_array = [2, 5, 7, 9, 3, 5, 6]
    # Read from contract
    response = await contract.duplicate_array(input_array).call()
    # Check the two arrays received are correct
    arr_1 = response.result.out_arr_1
    arr_2 = response.result.out_arr_2
    assert len(arr_1) == len(input_array)
    assert arr_1 == arr_2 == input_array
    # Check the values of the struct.
    calcs = response.result.array_calculation
    assert calcs.a == input_array[0] + input_array[1]
    assert calcs.b == calcs.a + input_array[2]
    assert calcs.c == calcs.b + input_array[3]
    assert calcs.d == calcs.c + input_array[4]

```
Run the test.
```
pytest test_array_returns.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy array_returns --alias array_returns
```

### Interact

Read-only
```
nile call array_returns duplicate_array 7 10 2 4 6 9 9 2
```
Returns:
```
7 10 2 4 6 9 9 2 12 16 22 31 7 10 2 4 6 9 9 2
^ Array          ^ Struct    ^ Array
```

### Public deployment

```
nile deploy array_returns --alias array_returns --network mainnet
```
```
🚀 Deploying array_returns
🌕 artifacts/array_returns.json successfully deployed to 0x07df4dcb06ad94f12c98fb344f4c621c95d47c79f0622f2c61dba76b497f6ba8
📦 Registering deployment as array_returns in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
