---
layout: page
title:  "Array as transaction argument"
permalink: /cairo/examples/array_argument/
toc: false
---

A call to an `@external` function may accept an array as an argument.

Below is a contract that accepts an array, performs a simple
calculation using the first and last elements and saves the result.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func stored_number() -> (res : felt):
end

# Function to get the stored value.
@view
func get{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        stored : felt
    ):
    let (stored) = stored_number.read()
    return (stored)
end

# Function to accept an array.
@external
func save{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        input_array_len : felt,
        input_array : felt*
    ):
    let first = input_array[0]
    let last = input_array[input_array_len - 1]
    let solution = first * 2 + last * 3
    stored_number.write(solution)
    return ()
end

```
Save as `contracts/array_argument.cairo`

### Compile

```
nile compile
```
Or compile this specific contract
```
nile compile contracts/array_argument.cairo
```

### Test

Make a new file called `test_array_argument.py` and populate it:

```py
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet

@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/array_argument.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    my_array = [2, 4, 6, 8, 9]
    # Save the array.
    # Notice how while the test function only expects the array,
    # and does not require the explicit passing of the array length.
    # The length is automatically passed to the Cairo function.
    await contract.save(my_array).invoke()

    # Read from contract
    response = await contract.get().call()
    # first * 2 + last * 3
    expected = my_array[0] * 2 + my_array[-1] * 3
    assert response.result.stored == expected
```
Run the test
```
pytest tests/test_array_argument.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy array_argument --alias array_argument
```

### Interact

Send an array to the contract. The length of the array must be passed
explicitly before the array itself. Note that in pytest this is handled
automatically.

Hence for an array: 1 2 3 4 10
The length is 5, and the arguments passed are: 5 1 2 3 4 10

Passing a non-5 number will cause an error.

```
nile invoke array_argument save 5 1 2 3 4 10
```
Read.
The contract returns `2*first + 3*last`
```
nile call array_argument get
```
Result: `32` as expected (`2 * 1 + 3 * 10`).


### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy array_argument --alias array_argument --network mainnet
```
Result:
```
🚀 Deploying array_argument
🌕 artifacts/array_argument.json successfully deployed to 0x00ebdf63de52e79a661bfc80faa361e7a96ed5675ac5f8a99944044bb27cfa6a
📦 Registering deployment as array_argument in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
