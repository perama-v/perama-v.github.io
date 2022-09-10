---
layout: page
title:  "Struct Returns"
permalink: /cairo/examples/struct_returns/
toc: false
---

A struct can be used to return values from an external function
in a contract. This might be useful for passing values between
two contracts compactly.

Below are two contracts:

- `struct_returns_UserDatabase.cairo` to store user data.
- `struct_returns_UserAnalyst.cairo` to score a user by polling the database.

The analyst is called with a user_id. It calls the database contract,
which responds with a struct. The analyst is aware of the struct definition
and can access the struct members. It then calculates the score
and returns that value.

`struct_returns_UserDatabase.cairo` below:
```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

struct User:
    member upvotes : felt
    member downvotes : felt
    member rank : felt
end

@storage_var
func user_upvotes(user_id : felt) -> (count : felt):
end

@storage_var
func user_downvotes(user_id : felt) -> (count : felt):
end

@storage_var
func user_rank(user_id : felt) -> (count : felt):
end

@external
func query_user{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        user_id : felt
    ) -> (
        user_stats : User
    ):
    let (up) = user_upvotes.read(user_id)
    let (down) = user_downvotes.read(user_id)
    let (rank) = user_rank.read(user_id)
    let user_stats = User(
        upvotes=up,
        downvotes=down,
        rank=rank)
    return (user_stats)
end

@external
func register_user{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        user_id : felt,
        upvotes : felt,
        downvotes : felt,
        rank : felt
    ):
    user_upvotes.write(user_id, upvotes)
    user_downvotes.write(user_id, downvotes)
    user_rank.write(user_id, rank)
    return ()
end
```
`struct_returns_UserAnalyst.cairo` below:
```sh
%lang starknet

from starkware.cairo.common.cairo_builtins import HashBuiltin
from contracts.struct_returns_UserDatabase import User

# This makes UserAnalyst (this contract) aware of UserDatabase.
@contract_interface
namespace UserDatabase:
    func query_user(user_id : felt) -> (user_stats : User):
    end
end

@view
func score_user{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        database_address : felt,
        user_id : felt
    ) -> (
        user_score : felt
    ):
    # Query stats of user
    let (user_stats) = UserDatabase.query_user(database_address, user_id)
    let user_score = user_stats.upvotes - user_stats.downvotes
    return (user_score)
end


```


### Compile

```
nile compile
```
Or compile this specific contract
```
nile compile contracts/struct_returns_UserAnalyst.cairo
nile compile contracts/struct_returns_UserDatabase.cairo
```

### Test

Make a new file called `test_struct_returns.py` and populate it:

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
    analyst = await starknet.deploy(
        "contracts/struct_returns_UserAnalyst.cairo")
    database = await starknet.deploy(
        "contracts/struct_returns_UserDatabase.cairo")
    return starknet, analyst, database

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, analyst, database = contract_factory

    USER_ID = 43
    UP = 200
    DOWN = 32
    RANK = 17
    # Call a function.
    await database.register_user(
        user_id=USER_ID,
        upvotes=UP,
        downvotes=DOWN,
        rank=RANK).invoke()

    # Ask the analyst contract to fetch-and-score the user.
    # It will receive a struct containing user data.
    response = await analyst.score_user(
        database.contract_address, USER_ID).call()

    assert response.result.user_score == UP - DOWN

    response = await database.query_user(user_id=USER_ID).invoke()
    # First capture the result of the call. In the contract the
    # name of the returned value is 'user_stats'.
    user = response.result.user_stats
    # The type of 'user_stats' is a a struct defined in the contract.
    # The struct member values can be accessed by name as follows.
    assert user.upvotes == UP
    assert user.downvotes == DOWN
    assert user.rank == RANK


```
Run the test
```
pytest tests/test_struct_returns.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy struct_returns_UserAnalyst --alias struct_returns_UserAnalyst
nile deploy struct_returns_UserDatabase --alias struct_returns_UserDatabase
```
Keep track of the addresses:
```
🚀 Deploying struct_returns_UserAnalyst
🌕 artifacts/struct_returns_UserAnalyst.json successfully deployed to 0x0154fe3ad44ef1cfd1b7b2a0cdbc31fb632b81ba990d1b72e48f16dbad563dd9
📦 Registering deployment as struct_returns_UserAnalyst in localhost.deployments.txt

🚀 Deploying struct_returns_UserDatabase
🌕 artifacts/struct_returns_UserDatabase.json successfully deployed to 0x05a5a2c002a8a3feaf3b416ae5b4cac541cc8672bab37bb560f343b53cbda5e9
📦 Registering deployment as struct_returns_UserDatabase in localhost.deployments.txt
```
### Interact

Create a user with ID=43, 200 upvotes, 32 downvoates and a rank of 17.
```
nile invoke struct_returns_UserDatabase register_user 43 200 32 17
```
Read. The analyst will score the user with the formula:
`upvotes - downvotes`.
```
nile call struct_returns_UserAnalyst score_user \
    0x05a5a2c002a8a3feaf3b416ae5b4cac541cc8672bab37bb560f343b53cbda5e9 \
    43
```
Result: `168`, as expected (`200 - 32`).

Now read the user data straight from the database. This will return
a struct.
```
nile call struct_returns_UserDatabase query_user 43
```
The returned values are the struct `User` struct with members in
the order in which they are defined (upvotes, downvotes, rank).
```
200 32 17
```

### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy struct_returns_UserAnalyst --alias struct_returns_UserAnalyst --network mainnet
nile deploy struct_returns_UserDatabase --alias struct_returns_UserDatabase --network mainnet
```
Results:
```
🚀 Deploying struct_returns_UserAnalyst
🌕 artifacts/struct_returns_UserAnalyst.json successfully deployed to 0x005cb4ad85ab48b6b11dfcd3bf33757f276b78ef74b6635c3b7b7487bbd9f6ca
📦 Registering deployment as struct_returns_UserAnalyst in mainnet.deployments.txt

🚀 Deploying struct_returns_UserDatabase
🌕 artifacts/struct_returns_UserDatabase.json successfully deployed to 0x061e90c9ee1657ef25fdf42a0356c45d05d08eb38e8bb626a9bd01b23d30bac4
📦 Registering deployment as struct_returns_UserDatabase in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online


