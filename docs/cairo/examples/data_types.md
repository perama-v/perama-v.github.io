---
layout: page
title:  "Data types"
permalink: /cairo/examples/data_types/
toc: false
---


There is only one data type in Cairo: a
[field element](https://www.cairo-lang.org/docs/how_cairo_works/cairo_intro.html), or `felt`.

A field element is best considered an unsigned 252-bit integer (uint252) for practical purposes.

Short strings up to 31 ASCII characters can be used, but they are represented internally
as `felt`s.

```sh
%lang starknet
%builtins range_check

@view
func echo_world(
        user_number : felt
    ) -> (
        user_number_echoed : felt,
        short_string : felt,
        mangled_string : felt,
        hello_string : felt,
    ):
    # Short strings are ASCII encoded humbers.
    # They are identified by single quotation marks.
    # They are actually just numbers (felts), not true strings.
    # 'ab' = a:61 b:62 = 0x6162 = 24930
    let short_string = 'ab'
    # Adding to a string does not make sense.
    # 24930 + 1 = 24931 = 0x6163 = a:61 c:63 = 'ac'
    let mangled_string = short_string + 1

    # h 68 e 65 l 6c l 6c o 6f
    # 0x68656c6c6f = 448378203247
    let hello_string = 'hello'
    return (user_number, short_string, mangled_string, hello_string)
end


```

Things to note:

- The new string representation is an ASCII-lookup conversion of two hex characters
to one ASCII character. Adding strings does not perform concatenation, it performs
addition of the underlying felts.

Save as `contracts/data_types.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/data_types.cairo
```

### Test

Make a new file called `test_data_types.py` and populate it:

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
    contract = await starknet.deploy("contracts/data_types.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Read from contract
    response = await contract.echo_world(10).call()
    assert response.result == (10, 24930, 24931, 448378203247)
```
Run the test
```
pytest tests/test_data_types.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy data_types --alias data_types
```

### Interact

Read
```
nile call data_types echo_world 10
```
Returns:
```
10 24930 24931 448378203247
```


### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy data_types --alias data_types --network mainnet
```

```
🚀 Deploying data_types
🌕 artifacts/data_types.json successfully deployed to 0x076ee2784726bf5a29edbcd8fdce5524416f0a06260ef83547d0151af1215d90
📦 Registering deployment as data_types in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
