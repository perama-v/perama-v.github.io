---
layout: page
title:  "Contract Calls"
permalink: /cairo/examples/contract_calls_A/
toc: false
---

One contract can call another. The contract making that call
must be made aware of the interface of the contract is is calling to.

```
# If A calls B:
contract_A.cairo <--- contains an interface function for B.
contract_B.cairo <--- not aware of A's existence.
```
Interfaces contain only the functions that are required. They
have the following format:

```
@contract_interface
namespace contract_B:
    func special_B_number(
    ) -> (
        number : felt
    ):
    end
end
```

Deploying two contracts that can read/write to each other requires
some checking of permissions. In this example contract B will not
update its number unless the request is coming from contract A.


Save the below as `contracts/contract_calls_A.cairo`
```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

# A stores a number.
@storage_var
func number_in_A() -> (res : felt):
end

# Address of contract B so that A can call B.
@storage_var
func contract_B_address() -> (res : felt):
end

# This makes A (this contract) aware of B.
# Basically copy the functions needed and strip out
# implicit arguments (inside the curly braces).
@contract_interface
namespace contract_B:
    func increment(
        number : felt
    ):
    end

    func read_number(
    ) -> (
        number : felt
    ):
    end
end

# Function to get the stored number.
@view
func get_AB_system_status{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        system_sum : felt
    ):
    # Sums A and B stored numbers.
    let (a_num) = number_in_A.read()
    # Fetch address of B from storage.
    let (b_addr) = contract_B_address.read()
    let (b_num) = contract_B.read_number(b_addr)
    let res = a_num + b_num
    return (res)
end

# Function to update the stored a number (field element).
@external
func update_AB_system{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        increment_for_A : felt,
        increment_for_B : felt,
    ):
    # Read A
    let (a_current) = number_in_A.read()
    # Add and save.
    number_in_A.write(a_current + increment_for_A)
    # Read B by calling it via the interface.
    let (b_addr) = contract_B_address.read()
    # This is the format for using interfaces.
    # contract_name.function(address, arg_1, arg_2, ...)
    contract_B.increment(b_addr, increment_for_B)
    return ()
end

# Save the address of contract B
@external
func set_B_address{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        address : felt
    ):
    # Save to storage in A (this contract).
    # Now A knows where to find B.
    # It is already aware of the nature of B, thanks to the
    # interface function.
    contract_B_address.write(address)
    return ()
end

```
Save the below as `contracts/contract_calls_B.cairo`
```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.syscalls import get_caller_address

# B stores a number.
@storage_var
func number_in_B() -> (res : felt):
end

@storage_var
func contract_A_address() -> (res : felt):
end

# Run on deployment only. Must have constructor in name and decorator.
@constructor
func constructor{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
      address_of_contract_A : felt
    ):
    # When this contract is deployed, passing it the known
    # address of A allows B to restrict write access.
    contract_A_address.write(address_of_contract_A)
    # There is no way to modify this value after deployment.
    return ()
end

# Function to get the stored number.
@view
func read_number{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        res : felt
    ):
    # Anyone can call this read-only function.
    let (res) = number_in_B.read()
    return (res)
end

# Function to update the stored number. Only A is allowed.
@external
func increment{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        input : felt
    ):
    # If using a local, 'alloc_locals' must go here.
    alloc_locals
    # See who is trying to modify B (this contract).
    let (local address) = get_caller_address()
    # Lookup the address of A saved during deployment.
    let (known_address) = contract_A_address.read()
    # Make sure it is A who is calling.
    assert address = known_address
    # Get the current stored number.
    let (current) = number_in_B.read()
    # Add and save.
    number_in_B.write(current + input)
    return ()
end
```

Things of note:

- Interface use is `contract_name.function(address, arg_1, arg_2, ...)`.
- The `get_caller_address()` is the core mechanism to enforce permissions.
- The `assert x = y` only works if `x` is a `local`. If not, `x` is remapped
to equal `y`.
    - `let x = 2`
    - `assert x = 3`. This redefines `x` to equal 3. Thus, references are
    not safe for checking permissions in this way. Use locals or an `if` statement
    `if x = 2:`
- Locals cannot be updated. They can be redefined:
    - `local x = 2`
    - `assert x = 2`. This checks the value equals 2.
    - `local x = 3`.
    - `assert x = 3`. This checks that `x` is now 3.
- The constructor is a good time to permanently save an address/parameter.
- Remember that `@view` is read and `@external` is write.


### Compile

```
nile compile
```
Or compile this specific contract
```
nile compile contracts/contract_calls_A.cairo
nile compile contracts/contract_calls_B.cairo
```

### Test

Make a new file called `test_contract_calls.py` and populate it:

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
    # Each contract is deployed.
    contract_A = await starknet.deploy(
        "contracts/contract_calls_A.cairo")
    # The address of A will be passed to B during deployment of B.
    addr_a = contract_A.contract_address
    # Pass the address so that B can assert that only A
    # has write-access to B's storage.
    contract_B = await starknet.deploy(
        "contracts/contract_calls_B.cairo",
        constructor_calldata=[addr_a])

    # Contract A needs to know where B is deployed so it can call B.
    await contract_A.set_B_address(contract_B.contract_address).invoke()
    # Now the main permissions are set, pass
    return starknet, contract_A, contract_B

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract_A, contract_B = contract_factory

    inc_A = 10
    inc_B = 99

    # Call contract A. It will update both A and B.
    await contract_A.update_AB_system(
            increment_for_A=inc_A,
            increment_for_B=inc_B).invoke()

    # Read from contract A. It will read from both A and B.
    response = await contract_A.get_AB_system_status().call()
    assert response.result.system_sum == inc_A + inc_B
```
Run the test
```
pytest tests/test_contract_calls.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy contract_calls_A --alias contract_calls_A
```
Result:
```
🚀 Deploying contract_calls_A
🌕 artifacts/contract_calls_A.json successfully deployed to 0x03e2b439c4dc4b6f44faa9f07b38b5cb32a8ac67e7cfe07c612fe8c1b6054b5b
📦 Registering deployment as contract_calls_A in localhost.deployments.txt
```
Take the address of A and pass it to B during deployment.
```
nile deploy contract_calls_B \
    0x03e2b439c4dc4b6f44faa9f07b38b5cb32a8ac67e7cfe07c612fe8c1b6054b5b \
    --alias contract_calls_B
```
Result:
```
🚀 Deploying contract_calls_B
🌕 artifacts/contract_calls_B.json successfully deployed to 0x06c4290b01e23bd585625c86e2b539525d062ee87f973f19df0851a015e32b76
📦 Registering deployment as contract_calls_B in localhost.deployments.txt
```

### Interact

Now that the address of B is known, save it in A. A knows where to call, B knows
who has permission to write.
```
nile invoke contract_calls_A set_B_address 0x06c4290b01e23bd585625c86e2b539525d062ee87f973f19df0851a015e32b76
```

```
nile invoke contract_calls_A update_AB_system 7 13
```
Read
```
nile call contract_calls_A get_AB_system_status
```
Result: 20

### Public deployment

Repeat the above deployment in the public testnet.

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy contract_calls_A --alias contract_calls_A --network mainnet
```
```
🚀 Deploying contract_calls_A
🌕 artifacts/contract_calls_A.json successfully deployed to 0x0501ace33a4a9b2f3dd4f4f5a4bf055a4cfec25ec143030efb4057cdb8acfbe5
📦 Registering deployment as contract_calls_A in mainnet.deployments.txt
```
```
nile deploy contract_calls_B \
    0x0501ace33a4a9b2f3dd4f4f5a4bf055a4cfec25ec143030efb4057cdb8acfbe5 \
    --alias contract_calls_B --network mainnet
```
```
🚀 Deploying contract_calls_B
🌕 artifacts/contract_calls_B.json successfully deployed to 0x02945e2e294bce105ff7a373cba13c18c1ba4dc6fe66bc649b848ff69ea97e37
📦 Registering deployment as contract_calls_B in mainnet.deployments.txt
```
Interact
```
nile invoke contract_calls_A set_B_address \
    0x02945e2e294bce105ff7a373cba13c18c1ba4dc6fe66bc649b848ff69ea97e37 \
    --network mainnet
```
```
Invoke transaction was sent.
Contract address: 0x0501ace33a4a9b2f3dd4f4f5a4bf055a4cfec25ec143030efb4057cdb8acfbe5
Transaction hash: 0x17e9322b65585cee575d5debb8c756544dcb3e2d15f97d44df72b65449a692d
```
```
nile invoke contract_calls_A update_AB_system 7 13 --network mainnet
```
```
Invoke transaction was sent.
Contract address: 0x0501ace33a4a9b2f3dd4f4f5a4bf055a4cfec25ec143030efb4057cdb8acfbe5
Transaction hash: 0x6eb5b46f538dc9506753e1cfc5aaa1cfd59b45c65011a2fe1493913368164e9
```

Read the system status:
```
nile call contract_calls_A get_AB_system_status --network mainnet
```
Result: `20`

Deployments can be viewed in the voyager explorer
https://voyager.online
