---
layout: page
title:  "Array"
permalink: /cairo/examples/array/
toc: false
---

Arrays are defined using a pointer. Their values are addressed by their location
in memory, relative to the pointer.

```sh
%lang starknet
%builtins range_check

from starkware.cairo.common.alloc import alloc

@view
func read_array{
        range_check_ptr
    }(
        index : felt
    ) -> (
        value : felt
    ):
    # A pointer to the start of an array.
    let (felt_array : felt*) = alloc()

    # [felt_array] is 'the value at the pointer'.
    # 'assert' sets the value at index
    assert [felt_array] = 9
    assert [felt_array + 1] = 8
    assert [felt_array + 2] = 7  # Set index 2 to value 7.
    assert [felt_array + 9] = 18
    # Access the list at the selected index.
    let val = felt_array[index]
    return (val)
end
```
Save as `array.cairo`.

### Compile

Compile all contracts
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/array.cairo
```

### Test

Make a new file called `array_test.py` and populate it:

```py
import pytest
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# Enables modules.
@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

# Reusable to save testing time.
@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/array.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Modify contract.
    await contract.read_array(9).invoke()

    # Read from contract
    val = await contract.read_array(9).call()
    assert val == 18
```

`pytest test_array.py`.

### Local Deployment

Deploy to the local devnet.
```
nile deploy array --alias array
```

### Interact
Read-only
```
nile call array FUNCTION_NAME ARG_1
```
Write
```
nile call array FUNCTION_NAME ARG_1

```

### Public deployment

```
nile deploy array --alias array --network mainnet
```
Deployments can be viewed in the voyager explorer
https://voyager.online
