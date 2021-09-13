---
layout: page
title:  "Unit tests"
permalink: /cairo/examples/unit_test/
toc: false
---

A contract can be tested before deployment.

Below is a contract that stores pen and paper inventory across
different warehouses. To test that the contract works as expected
a test is created that stores to the inventory twice with `record_items()`,
then calls `check_items()`. By asserting that the values received match those
that are expected, the test identifies errors before deployment.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

# Define a mapping for storage (pen ID 1, paper ID 2).
# (warehouse num, item num in -> count out).
@storage_var
func inventory(warehouse_number : felt,
    item_id : felt) -> (count : felt):
end

# Changes inventory, adding elements to a specific warehouse.
@external
func record_items{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(
        warehouse_number : felt, pens : felt, paper : felt):
    # Ge the current inventory for the specified warehouse.
    let (current_pens) = inventory.read(
        warehouse_number=warehouse_number, item_id=1)
    let (current_paper) = inventory.read(
        warehouse_number=warehouse_number, item_id=2)
    # Add new to old values.
    inventory.write(warehouse_number=warehouse_number,
        item_id=1, value=pens + current_pens)
    inventory.write(warehouse_number=warehouse_number,
        item_id=2, value=paper + current_paper)
    return ()
end

# Returns the inventory state.
@view
func check_items{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(warehouse_number : felt) -> (
        pens : felt, paper : felt):
    let (current_pens) = inventory.read(
        warehouse_number=warehouse_number, item_id=1)
    let (current_paper) = inventory.read(
        warehouse_number=warehouse_number, item_id=2)
    return (current_pens, current_paper)
end


```
Save as `test_inventory.cairo`.

### Test

Make a new file called `contract_test.py` and populate it:

```py
import os
import pytest

from starkware.starknet.compiler.compile import (
    compile_starknet_files)
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# The path to the contract source code.
CONTRACT_FILE = os.path.join(
    os.path.dirname(__file__), "test_inventory.cairo")


# The testing library uses python's asyncio. So the following
# decorator and the ``async`` keyword are needed.
@pytest.mark.asyncio
async def test_record_items():
    # Compile the contract.
    contract_definition = compile_starknet_files(
        [CONTRACT_FILE], debug_info=True)

    # Create a new Starknet class that simulates the StarkNet
    # system.
    starknet = await Starknet.empty()

    # Deploy the contract.
    contract_address = await starknet.deploy(
        contract_definition=contract_definition)
    contract = StarknetContract(
        starknet=starknet,
        abi=contract_definition.abi,
        contract_address=contract_address,
    )

    # Store pens and paper twice.
    await contract.record_items(
        warehouse_number=3, pens=45, paper=10).invoke()
    await contract.record_items(
        warehouse_number=3, pens=55, paper=100).invoke()

    # Check the result of check_items().
    assert await contract.check_items(
        warehouse_number=3).call() == (100, 110)
```

The packages required for testing are installed by default with cairo-lang.
Run the test:

`pytest contract_test.py`.

Result:
```
=== 1 passed, 1 warning in 13.42s ===
```

### Compile

Then, to compile and send to StarkNet:
```
starknet-compile test_inventory.cairo \
    --output test_inventory_compiled.json \
    --abi test_inventory_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract test_inventory_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x07681895f0eca80d71bd4bd5e050b55f180be63b1544558d20aa39e81ddb55e9
Transaction ID: 123329
```

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=123329

Returns:
{
    "block_id": 13349,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/13349) and the
[contract](https://voyager.online/contract/0x07681895f0eca80d71bd4bd5e050b55f180be63b1544558d20aa39e81ddb55e9)

### Interact

Then, to interact:

```
starknet invoke \
    --network=alpha \
    --address 0x07681895f0eca80d71bd4bd5e050b55f180be63b1544558d20aa39e81ddb55e9 \
    --abi test_inventory_contract_abi.json \
    --function record_items \
    --inputs 3 30 40

Returns:
Invoke transaction was sent.
Contract address: 0x07681895f0eca80d71bd4bd5e050b55f180be63b1544558d20aa39e81ddb55e9
Transaction ID: 123380
```

Then check the value:
```
starknet call \
    --network=alpha \
    --address 0x07681895f0eca80d71bd4bd5e050b55f180be63b1544558d20aa39e81ddb55e9 \
    --abi test_inventory_contract_abi.json \
    --function check_items \
    --inputs 3


Returns:
30 40
```

The live contract matches our testing: 30 pencil, 40 paper are recorded.

### Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.