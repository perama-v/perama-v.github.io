---
layout: page
title:  "Read and write a tuple"
permalink: /cairo/examples/read_write_tuple/
toc: false
---

Data may be stored as a tuple of values.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

# The tuple to be stored is an output of the storage function.
@storage_var
func stored_felt() -> (res : (felt, felt, felt)):
end

# Function to get the stored tuple.
@view
func get{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        res_1 : felt,
        res_2 : felt,
        res_3 : felt
    ):
    let (stored_tuple) = stored_felt.read()
    let res_1 = stored_tuple[0]
    let res_2 = stored_tuple[1]
    let res_3 = stored_tuple[2]
    return (res_1=res_1, res_2=res_2, res_3=res_3)
end

# Function to update the stored tuple of field elements.
@external
func save{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        input_1 : felt,
        input_2 : felt,
        input_3 : felt
    ):
    # The tuple is declared with round brackets.
    stored_felt.write((input_1, input_2, input_3))
    return ()
end

```
Save as `contracts/read_write_tuple.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/read_write_tuple.cairo
```

### Test

Make a new file called `test_read_write_tuple.py` and populate it:

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
    contract = await starknet.deploy("contracts/read_write_tuple.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Modify contract.
    await contract.save(4, 67, 97).invoke()

    # Read from contract
    response = await contract.get().call()
    assert response.result == (4, 67, 97)
```
Run the test
```
pytest tests/test_read_write_tuple.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy read_write_tuple --alias read_write_tuple
```

### Interact

Write
```
nile invoke read_write_tuple save 23 44 66
```
Read
```
nile call read_write_tuple get
```
Returns:
```
23 44 66
```

### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy read_write_tuple --alias read_write_tuple --network mainnet
```
Returns:
```
🚀 Deploying read_write_tuple
🌕 artifacts/read_write_tuple.json successfully deployed to 0x04d618437892af8e3418ac3c79d73d543c26ec6f35d43767866fdbd58bc65b0f
📦 Registering deployment as read_write_tuple in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
