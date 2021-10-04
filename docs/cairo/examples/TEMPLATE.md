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
Save as `TEMPLATE.cairo`.


### Compile

Then, to compile:
```
starknet-compile TEMPLATE.cairo \
    --output TEMPLATE_compiled.json \
    --abi TEMPLATE_contract_abi.json
```

### Test

Make a new file called `TEMPLATE_contract_test.py` and populate it:

```py
import pytest
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

@pytest.mark.asyncio
async def test_database():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("TEMPLATE.cairo")

    # Call a function.
    await contract.FUNCTION_NAME(
        INPUT_1, INPUT2).invoke()

    # Check the result of a function.
    VAL = await contract.FUNCTION_NAME().call()
    assert VAL == EXPECTED_RES
```

The packages required for testing are installed by default with cairo-lang.
Run the test:

`pytest TEMPLATE_contract_test.py`.

Check that the tests passed:
```
=== 1 passed, 1 warning in xx.xxs ===
```

### Deploy

Then, to deploy:
```
starknet deploy --contract TEMPLATE_compiled.json \
    --network=alpha

Returns:
RESULT
```

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=ID

Returns:
RESULT
```
The [block](https://voyager.online/block/ID) and the
[contract](https://voyager.online/contract/ADDRESS)

### Interact

Then, to interact:


```
starknet invoke \
    --network=alpha \
    --address ADDRESS \
    --abi TEMPLATE_contract_abi.json \
    --function FUNCTION_NAME \
    --inputs INPUT1 INPUT2

Returns:
RESULT
```

### Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.