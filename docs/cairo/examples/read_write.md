---
layout: page
title:  "Read and write"
permalink: /cairo/examples/read_write/
toc: false
---

To write to a state variable, you need to send a transaction. This is stored in StarkNet
and is included as rollup data on Ethereum.

To read a state variable, no transaction is needed.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func stored_felt() -> (res : felt):
end

# Function to get the stored number.
@view
func get{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        res : felt
    ):
    let (res) = stored_felt.read()
    return (res)
end

# Function to update the stored number (field element).
@external
func save{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        input : felt
    ):
    stored_felt.write(input)
    return ()
end

```
Save as `contracts/read_write.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/read_write.cairo
```

### Test

Make a new file called `test_read_write.py` and populate it:

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
    contract = await starknet.deploy("contracts/read_write.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Modify contract.
    await contract.save(10009).invoke()

    # Read from contract
    response = await contract.get().call()
    assert response.result.res == 10009
```
Run the test
```
pytest tests/test_read_write.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy read_write --alias read_write
```

### Interact

Write
```
nile invoke read_write save 101017
```
Read
```
nile call read_write get
```
Returns: `101017`

### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy read_write --alias read_write --network mainnet
```
Deployments can be viewed in the voyager explorer
https://voyager.online
