---
layout: page
title:  "TEMPLATE"
permalink: /cairo/examples/TEMPLATE/
toc: false
---

DESCRIPTION

```sh
CONTRACT

```
Save as `contracts/TEMPLATE.cairo`

### Compile

Compile all contracts
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/TEMPLATE.cairo
```

### Test

Make a new file called `test_TEMPLATE.py` and populate it:

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
    contract = await starknet.deploy("contracts/TEMPLATE.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Modify contract.
    await contract.EXTERNAL_FUNCTION_NAME(INPUT_1, INPUT2).invoke()

    # Read from contract
    VAL = await contract.VIEW_FUNCTION_NAME().call()
```
Run the test
```
pytest test_TEMPLATE.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy TEMPLATE --alias TEMPLATE
```

### Interact
Read-only
```
nile call TEMPLATE FUNCTION_NAME ARG_1
```
Write
```
nile invoke TEMPLATE FUNCTION_NAME ARG_1

```

### Public deployment

```
nile deploy TEMPLATE --alias TEMPLATE --network mainnet
```
Deployments can be viewed in the voyager explorer
https://voyager.online
