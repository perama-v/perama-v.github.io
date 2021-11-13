---
layout: page
title:  "Hello, World!"
permalink: /cairo/examples/hello_world/
toc: false
---

The Cairo comiler knows that a program is a contract
if it has `%lang starknet` at the top.

In Cairo, each number is a `felt` (field element -> "F EL T").
This is an interesting topic to explore. In practice, field elements
behave like integers inside StarkNet contracts.

```sh
%lang starknet
%builtins range_check

# Alphabet substituation cipher for each letter.
# a = 01, b = 02, etc.
const hello = 10000805121215  # 08, 05, 12, 12, 15.
const world = 10002315181204  # 23, 15, 18, 12, 04.

@view
func greeting() -> (number_1 : felt, number_2 : felt):
    return (hello, world)
end
```

Things of note

- The `range_check` is a component that Cairo uses to make sure
numbers stay within a certain range.
- The `@view` decorator exposes the `greeting` function for reading.
- The function accepts no inputs
- The function returns two outputs.
- The two outputs have the type `felt`.
- All functions must have a `return` and and `end`

Save as `contracts/hello_world.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/hello_world.cairo
```

### Test

Make a new file called `test_hello_world.py` and populate it:

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
    contract = await starknet.deploy("contracts/hello_world.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Read from contract
    response = await contract.greeting().call()
    assert response.result == (
        10000805121215,
        10002315181204)
```
Run the test
```
pytest test_hello_world.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy hello_world --alias hello_world
```

### Interact

Read-only
```
nile call hello_world greeting
```
Returns the two hexadecimal numbers:
```
0x9187e6fccbf 0x918d8717c94
```
Which when converted to decimal using python are confirmed:
```py
>>> int('0x9187e6fccbf', 16)
10000805121215
```

### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy hello_world --alias hello_world --network mainnet
```
Returns:
```
🚀 Deploying hello_world
🌕 artifacts/hello_world.json successfully deployed to 0x01ecf2883c6e8b130e4c092d46e7c36db3fe38364e885d7110a9acb6172b6235
📦 Registering deployment as hello_world in mainnet.deployments.txt
```

Deployments can be viewed in the voyager explorer
https://voyager.online
