---
layout: page
title:  "struct_arguments"
permalink: /cairo/examples/struct_arguments/
toc: false
---

A struct can be used to return values from an external function
in a contract. This might be useful for passing values between
two contracts compactly.

Below are two contracts:

- `UserDatabase.cairo` to store user data.
- `UserAnalyst.cairo` to score a user by polling the database.

The analyst is called with a user_id. It calls the database contract,
which responds with a struct. The analyst is aware of the struct definition
and can access the struct members. It then calculates the score
and returns that value.

`UserDatabase.cairo` below:
```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

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
        storage_ptr : Storage*,
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
        storage_ptr : Storage*,
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
`UserAnalyst.cairo` below:
```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

# The struct being received is also defined in the receiving contract.
struct User:
    member upvotes : felt
    member downvotes : felt
    member rank : felt
end

# The interface for the other function is defined.
@contract_interface
namespace IUserDatabase:
    func query_user(
        user_id : felt
    ) -> (
        user_stats : User
    ):
    end
end

@external
func score_user{
        syscall_ptr : felt*,
        storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        database_address : felt,
        user_id : felt
    ) -> (
        user_score : felt
    ):
    let (user : User) = IUserDatabase.query_user(
        database_address, user_id)
    let score = user.upvotes - user.downvotes
    return (score)
end

```

### Test

Make a test file called `struct_arguments_contract_test.py`:

```py
import pytest
from starkware.starknet.testing.starknet import Starknet

@pytest.mark.asyncio
async def test_database():
    starknet = await Starknet.empty()
    database = await starknet.deploy("UserDatabase.cairo")
    analyst = await starknet.deploy("UserAnalyst.cairo")

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
    user_score = await analyst.score_user(
        database.contract_address, USER_ID)
    assert user_score == UP - DOWN

    # pytest framework does not allow fetching of a struct directly.
    # FAILS with:
    # 'TypeError: Return argument user_stats expected to be
    # of type felt. Got: User.'

    #user = await contract.query_user(user_id=USER_ID).invoke()
    #assert user.upvotes == UP
    #assert user.downvotes == DOWN
    #assert user.rank == RANK
```

`pytest struct_arguments_contract_test.py`.

Check that the tests passed:

TODO: Investigate the error:
```
>       analyst = await starknet.deploy("UserAnalyst.cairo")
AssertionError: Type is expected to be fully resolved at this point.
```

TODO: Investigate the error when pytest tries to receive `User`
directly. Caused by the commented-out test code.
```
TypeError: Return argument user_stats expected to be of type felt. Got: User.
```


