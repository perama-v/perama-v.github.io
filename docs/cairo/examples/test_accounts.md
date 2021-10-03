---
layout: page
title:  "Test Accounts"
permalink: /cairo/examples/test_accounts/
toc: false
---

In StarkNet, accounts are contracts. An account contract executes a transaction
when it is instructed to do so. This is an exploration of how accounts
are deployed for use, and how testing might work in this model.

For background information on the abstraction of accounts see [**here**]({{ site.baseurl }}{% link cairo/account_abstraction.md %}). In this example, the account
is a simple single owner account, where a single ECDSA signature can be used to
authorize the execution of a transaction.

The steps a user takes to interact with an application are:

- Deploy account contract.
- Initialize the account with their ECDSA public key.
- Gather details about the application contract (e.g., call a particular function)
- Use their private key to sign the transaction details
- Pass the details and signature to their account contract.
    - The account contract checks the signature.
    - Then calls the specified function on application contract.

This `pytest`-based framework is structured like so:

- A blank, local `StarkNet` object is created
- An `Account.cairo` contract is chosen.
    - I have chosen the OpenZeppelin contract
[here](https://github.com/OpenZeppelin/cairo-contracts/blob/main/contracts/Account.cairo)
and have added a `get_nonce()` function.
- An `Account` object is created, using a private key and fallback L1
address of the user.
    - This generates a `Signer` object is created using the private
    key of the user.
    - This produces a public key.
- The `Account.create()` method is called. This:
    1. Compiles and deploys the account contract
    2. Calls initialize function on the contract, saving
    the public key into the account. This secures the account.
- The details of the application contract are obtained. Here this means:
    - The `Application.cairo` contract is deployed and it's address obtained.
- The `Account.tx_with_nonce()` method is called. This:
    1. Calls the `get_nonce()` method of the Account.
    2. Uses the signer to build the transaction and sign it.
    3. Third, calls the `execute()` function on the account contract
        - The account checks the signature.
        - Then calls the specified application contract with the transaction details.
- The account, being a contract on the simulated starknet, has a stored nonce
that auto-increments.

The purpose of the `Account` utility module is to allow the nonce management to
be hidden, and to allow accounts to be created simply. E.g., a list of accounts
where the first is an admin, and the rest are different users.

Project structure:

```sh
├── contracts
│   ├── Account.cairo
│   └── Application.cairo
└── tests
    ├── Application_test.py
    └── utils
        ├── Account.py
        └── Signer.py
```
In addition to the `Account.py` (my addtion), and `Signer.py` (by Martín in OpenZeppelin cairo-contracts) the main body of the test framework is from the
StarkWare library that is installed with `pip install cairo-lang`. It can
be viewed
[here](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/starknet/testing/starknet.py).

The application in this example is a contract that stores a personal number.
It associates a the number with the contract (account) that calls the function.

Any contract (account) can store a number - however - nobody can imitate another
identity. The application contract read the caller address at the protocol level.

Thus, if you have an account contract, you are the only person that can store
a number for your account. This example could be expanded to include a different
account model, e.g., a `MultiSig.cairo` account that handles signatures from multiple
parties. The application contract could already be deployed and would handle this
new contract as it would this simple singl owner `Account.cairo`. For an account is an
abstract concept from the perspective of the application.

Run the test command:

```sh
pytest -s tests/Application_test.py
```
This generates two accounts. Each account is used to store
a number in the Application
See that account contracts are being deployed:
```sh
tests/Application_test.py Deploying 2 accounts...
Account 0 is: Account with address 0x5808...31d0 and signer public key 0x4024...c82b
Account 1 is: Account with address 0x4f9a...7d08 and signer public key 0x31f4...4791
```
See that two accounts are created. Note that the public key (ECDSA) that the account
contract uses is different from the account address. An account address in StarkNet is
the address of the deployed account contract.

---

## Application

This is the contents of the `Application.cairo` file.

Once deployed, can be called by anyone to publicly store a number.
E.g., a personal annoucements board where you can signal support
for some numbered-thing ('I support xyz number 554').

Nobody can impersonate you or change/conceal your announcement.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage
from starkware.starknet.common.syscalls import get_caller_address

# Storage.
@storage_var
func stored_number(account_id : felt) -> (res : felt):
end

# Anyone can save their personal number.
@external
func store_number{
        storage_ptr : Storage*,
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        number_to_store : felt
    ):
    # Fetch the address of the contract that called this function.
    let (account_address) = get_caller_address()
    stored_number.write(account_address, number_to_store)
    return ()
end

# Anyone can view the number for any address.
@view
func view_number{
        storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        account_address : felt
    ) -> (
        stored_num : felt
    ):
    let (stored_num) = stored_number.read(account_address)
    return (stored_num)
end
```

---

## Testing

This is the contents of the `Application_test.py` file.

```py
# Requires cairo-lang >= 0.4.1
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet
from utils.Account import Account

# Create signers that use a private key to sign transaction objects.
NUM_SIGNING_ACCOUNTS = 2
DUMMY_PRIVATE = 123456789987654321
# All accounts currently have the same L1 fallback address.
L1_ADDRESS = 0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984

@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

@pytest.fixture(scope='module')
async def account_factory():
    # Initialize network
    starknet = await Starknet.empty()
    accounts = []
    print(f'Deploying {NUM_SIGNING_ACCOUNTS} accounts...')
    for i in range(NUM_SIGNING_ACCOUNTS):
        account = Account(DUMMY_PRIVATE + i, L1_ADDRESS)
        await account.create(starknet)
        accounts.append(account)
        print(f'Account {i} is: {account}')

    # Admin is usually accounts[0], user_1 = accounts[1].
    # To build a transaction to call func_xyz(arg_1, arg_2)
    # on a TargetContract:

    # user_1 = accounts[1]
    # await user_1.tx_with_nonce(
    #     to=TargetContractAddress,
    #     selector_name='func_xyz',
    #     calldata=[arg_1, arg_2])
    return starknet, accounts


@pytest.fixture(scope='module')
async def application_factory(account_factory):
    starknet, accounts = account_factory
    application = await starknet.deploy("contracts/application.cairo")
    return starknet, accounts, application

@pytest.mark.asyncio
async def test_store_number(application_factory):
    _, accounts, application = application_factory
    # Let two different users save a number.
    user_0 = accounts[0]
    user_0_number = 543
    user_1 = accounts[1]
    user_1_number = 888

    await user_0.tx_with_nonce(
        to=application.contract_address,
        selector_name='store_number',
        calldata=[user_0_number])

    await user_1.tx_with_nonce(
            to=application.contract_address,
            selector_name='store_number',
            calldata=[user_1_number])

    # View transactions don't require an authorized transaction.
    (user_0_stored, ) = await application.view_number(
        user_0.address).invoke()
    (user_1_stored, ) = await application.view_number(
        user_1.address).invoke()

    assert user_0_stored == user_0_number
    assert user_0_stored == user_0_number
```
---

## Account helper module

This is the contents of the `Account.py` file.

```py
from utils.Signer import Signer

ACCOUNT_PATH = "contracts/Account.cairo"

# A deployed contract-based account with nonce awareness.
class Account():
    # Initialize with signer. Deploy separately.
    def __init__(self, private_key, L1_address):
        self.signer = Signer(private_key)
        self.L1_address = L1_address
        self.address = 0
        self.contract = None

    def __str__(self):
        addr = hex(self.address)
        pubkey = hex(self.signer.public_key)
        return f'Account with address {addr[:6]}...{addr[-4:]} and \
signer public key {pubkey[:6]}...{pubkey[-4:]}'

    # Deploy. Creates a contract and initializes.
    async def create(self, starknet):
        contract = await starknet.deploy(ACCOUNT_PATH)
        self.contract = contract
        self.address = contract.contract_address
        await contract.initialize(self.signer.public_key,
            self.L1_address).invoke()

    # Transact. Signs and sends a transaction using latest nonce.
    async def tx_with_nonce(self, to, selector_name, calldata):
        (nonce, ) = await self.contract.get_nonce().call()
        transaction = await self.signer.build_transaction(
            account=self.contract,
            to=to,
            selector_name=selector_name,
            calldata=calldata,
            nonce=nonce
        ).invoke()
        return transaction
```

---

## Signer helper module

This is the contents of the `Signer.py` file.

The `build_transaction()` is used by the account object to
to call the `execute()` function in the account contract with
a specific transaction payload.

```py
from starkware.crypto.signature.signature import (
    pedersen_hash, private_to_stark_key, sign)
from starkware.starknet.public.abi import get_selector_from_name


class Signer():
    def __init__(self, private_key):
        self.private_key = private_key
        self.public_key = private_to_stark_key(private_key)

    def sign(self, message_hash):
        return sign(msg_hash=message_hash,
            priv_key=self.private_key)

    def build_transaction(self, account, to, selector_name,
            calldata, nonce):
        selector = get_selector_from_name(selector_name)
        message_hash = hash_message(to, selector, calldata, nonce)
        (sig_r, sig_s) = self.sign(message_hash)
        return account.execute(to, selector, calldata, nonce, sig_r,
            sig_s)


def hash_message(to, selector, calldata, nonce):
    res = pedersen_hash(to, selector)
    res_calldata = hash_calldata(calldata)
    res = pedersen_hash(res, res_calldata)
    return pedersen_hash(res, nonce)


def hash_calldata(calldata):
    if len(calldata) == 0:
        return 0
    elif len(calldata) == 1:
        return calldata[0]
    else:
        return pedersen_hash(hash_calldata(calldata[1:]),
            calldata[0])
```

---

## Account contract

This is the contents of the `Account.cairo` file.

The `get_nonce()` method was added to the
[original file](https://github.com/OpenZeppelin/cairo-contracts/blob/main/contracts/Account.cairo) (commit hash `831bc0f`, license MIT).


```sh
%lang starknet
%builtins pedersen range_check ecdsa

from starkware.cairo.common.hash import hash2
from starkware.cairo.common.registers import get_fp_and_pc
from starkware.cairo.common.signature import verify_ecdsa_signature
from starkware.cairo.common.cairo_builtins import (HashBuiltin,
    SignatureBuiltin)
from starkware.starknet.common.syscalls import call_contract
from starkware.starknet.common.storage import Storage

struct Message:
    member to: felt
    member selector: felt
    member calldata: felt*
    member calldata_size: felt
    member nonce: felt
end

struct SignedMessage:
    member message: Message*
    member sig_r: felt
    member sig_s: felt
end

@storage_var
func current_nonce() -> (res: felt):
end

@storage_var
func public_key() -> (res: felt):
end

@storage_var
func initialized() -> (res: felt):
end

@storage_var
func L1_address() -> (res: felt):
end

@external
func initialize{
        storage_ptr: Storage*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    } (_public_key: felt, _L1_address: felt):
    let (_initialized) = initialized.read()
    assert _initialized = 0
    initialized.write(1)

    public_key.write(_public_key)
    L1_address.write(_L1_address)
    return ()
end

func hash_message{
        pedersen_ptr : HashBuiltin*
    }(  message: Message*
    ) -> (
        res: felt
    ):
    alloc_locals
    let (res) = hash2{hash_ptr=pedersen_ptr}(message.to,
        message.selector)
    # we need to make `res` local
    # to prevent the reference from being revoked
    local res = res
    let (res_calldata) = hash_calldata(message.calldata,
        message.calldata_size)
    let (res) = hash2{hash_ptr=pedersen_ptr}(res, res_calldata)
    let (res) = hash2{hash_ptr=pedersen_ptr}(res, message.nonce)
    return (res=res)
end

func hash_calldata{
        pedersen_ptr: HashBuiltin*
    }(
        calldata: felt*,
        calldata_size: felt
    ) -> (
        res: felt
    ):
    if calldata_size == 0:
        return (res=0)
    end

    if calldata_size == 1:
        return (res=[calldata])
    end

    let _calldata = [calldata]
    let (res) = hash_calldata(calldata + 1, calldata_size - 1)
    let (res) = hash2{hash_ptr=pedersen_ptr}(res, _calldata)
    return (res=res)
end

func validate{
        storage_ptr: Storage*,
        pedersen_ptr: HashBuiltin*,
        ecdsa_ptr: SignatureBuiltin*,
        range_check_ptr
    } (
        signed_message: SignedMessage*
    ):
    alloc_locals

    # validate nonce
    let (_current_nonce) = current_nonce.read()
    assert _current_nonce = signed_message.message.nonce

    # reference implicit arguments to prevent them from being
    # revoked by `hash_message`
    local storage_ptr : Storage* = storage_ptr
    local range_check_ptr = range_check_ptr

    # verify signature
    let (message) = hash_message(signed_message.message)
    let (_public_key) = public_key.read()

    verify_ecdsa_signature(
        message=message,
        public_key=_public_key,
        signature_r=signed_message.sig_r,
        signature_s=signed_message.sig_s)

    return ()
end

@external
func execute{
        storage_ptr: Storage*,
        pedersen_ptr: HashBuiltin*,
        ecdsa_ptr: SignatureBuiltin*,
        syscall_ptr: felt*,
        range_check_ptr
    } (
        to: felt,
        selector: felt,
        calldata_len: felt,
        calldata: felt*,
        nonce: felt,
        sig_r: felt,
        sig_s: felt
    ) -> (
        response : felt
        ):
    alloc_locals

    let (__fp__, _) = get_fp_and_pc()
    local message: Message = Message(
        to, selector, calldata, calldata_size=calldata_len, nonce)
    local signed_message: SignedMessage = SignedMessage(
        &message, sig_r, sig_s)

    # validate transaction
    validate(&signed_message)

    # bump nonce
    let (_current_nonce) = current_nonce.read()
    current_nonce.write(_current_nonce + 1)

    # execute call
    let response = call_contract(
        contract_address=message.to,
        function_selector=message.selector,
        calldata_size=message.calldata_size,
        calldata=message.calldata
    )

    return (response=response.retdata_size)
end

#####
# Getters
###

@view
func get_public_key{
        storage_ptr: Storage*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }() -> (
        res: felt
    ):
    let (res) = public_key.read()
    return (res=res)
end

@view
func get_L1_address{
        storage_ptr: Storage*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }() -> (
        res: felt
    ):
    let (res) = L1_address.read()
    return (res=res)
end


# New.
@view
func get_nonce{
        storage_ptr: Storage*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }() -> (
        res: felt
    ):
    let (res) = current_nonce.read()
    return (res=res)
end
```
---

## Notes

The `tx_with_nonce()` method in `Account.py` does not currently
handle arrays in calldata. It is likely related to the
requirement for the length of the array to be passed to the function
prior to the array itself.