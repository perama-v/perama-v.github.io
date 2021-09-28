---
layout: page
title:  "Accounts on StarkNet"
permalink: /cairo/account-abstraction/
toc: false
---

An account is an identity that persists across applications on a blockchain.

A familiar model is the Ethereum account model, comprising a long (`0x123abc456def...`) number
and a nonce, which increases every time you send a transaction. The account is derived from
a private key, perhaps derived from a mnemonic seed using wallet software.

Send something "from your account" on Ethereum, you use the private key to sign a message that
effectively says "I control this account, and the next transaction for this account is xyz".
The network can verify your signature matches the publicly visible account address and
the transaction is deemed "valid for inclusion".

### Token transfer (*concrete* accounts)

- If someone sends a token to your account, they interact with the token contract to change the
recorded owner from their address to your address.
- The token contract performs a check that `the identity (key) used to sign the transaction to make this transfer`
is recorded as the current owner of the token.

In StarkNet, the account model is different. It is a more flexible model, and one that
the Ethereum community desired for a long time, as this
[excellent article](https://our.status.im/account-abstraction-eip-2938/) explores.

```
TL;DR

Tired (concrete): Check that transaction comes with a correctly signed signature
for the given address.

Wired (abstract): Check that the transaction comes from the given address.
```

### Token transfer (*abstract* accounts)

- If someone sends a token to your account, they interact with the token contract to change the
recorded owner from their address to your address.
- The token contract performs a check that `the identity (contract) used to make this transfer`
is recorded as the current owner of the token.

With abstract accounts, it's the **who** (address) that matters, not the **how** (signature).

## Contracts are accounts

The concept of an account is now a `contract_address`. A StarkNet user, will still
have an address that can be shared publicly so that someone can send a token to that address.
The user still has a wallet with private keys that are used to sign transactions.

The difference is that the address will be a contract. The contract can contain any code.
Below is an example of a stub contract that could be used as an account. The user deploys this
contract and calls `initialize()`, storing the public key that their wallet generated.

Then to transfer a token, they put together the details (token quantity, recipient) and
sign the message with their wallet. Then `execute()` is called, passing the signed message
and intended destination (the address of the token contract).

```sh
### A minimalist Account contract ###

# This function is called once to set up the account contract.
@external
func initialize(public_key):
    store_public_key(public_key)
    return ()
end

# This function is called for every transaction the user makes.
@external
func execute(destination, transaction_data, signature):
    # Check the signature used matches the one stored.
    check_signature(transaction_data, signature)
    # Increment the transaction counter for safety.
    increase_nonce()
    # Call the specified destination contract.
    call_destination(destination, transaction_data)
    return ()
end
```

For an implementation of this model see
[here](https://github.com/OpenZeppelin/cairo-contracts/blob/main/contracts/Account.cairo).

A user can define what they want their account to "be". For many users,
an account contract sill perform a signature check and then call the destination.

However, the contract may do anything:

- Check multiple signatures.
- Receive and swap a stable coin before paying for the transaction fee.
- Only agree to pay for the transaction if a trade was profitable.
- Use funds from a mixer to pay for the transaction, separating the deposit from withdrawl.
- Send a transaction on behalf of itself - a DAO.

## Abstraction terminology

A token contract maintains a precise records of token owners. These owners are all
contract addresses, and it is not important to know what sort of code is at that contract.
It simply operates under the model of "if an address was specified as the subject of
a token operation, the agent must have thought about how they are taking care of ownership
models and signature checks". In this way, from the perspective of the token contract,
and account is an abstract concept - it could be a DAO or a multisig or anything imaginable.

## L1 to L2 Onboarding to a new account

To make a new account, a contract must be deployed. A contract may be selected from a library
of well-scutinized contracts for different use cases. E.g,. Single-owner, multi-owner, or
more specialised variants such as nonce-less accounts for exotic pruposes.

A user will likely experience an interface that combines key elements involved in onboarding to L2:

- Bringing payment from L1 (to pay for transactions).
- Generating a StarkNet key using their wallet (E.g., Specific ECDSA curve, StarkNet-specific
Hierarchical Deterministic path).
- Selection of the type of account desired (E.g., simple single owner account).
- Information that the selected account requires (E.g., Specifying an L1 address they want to
accept funds in the event of a forced closure of L2. Or specifying the list of the multiple
owners for the account).

The wallet is used to sign a transaction to deploy the contract and save their details.

The Account may then be used for all interactions with StarkNet, such as transferring, swapping
or caling arbitrary contracts with arbitrary data.

This may exist as a single operation as part of an L1 to L2 bridge. The bridge might receive the
L1 funds and use those to deploy and initialize a contract using the information specified in
the bridge transfer. E.g., A solidity contract with `bridgeToNewAccount(L2_StarkNetKey,
L1_FallbackAddress, etherAmount)`. The address of the deployed account contract could then be
fetched and saved for the user for their first L2 transaction.


Specific example
================

This is an outline for a minimum viable StarkNet application with user authentification with account abstraction.

## Application design

Consider a contract that stores a number for a user. Anyone can save a number, but
only for themselves. The contract maintains the identity of each user by associating the
`account_address` with the `stored_number`.

The contract has a strict requirement that the number saved is associated with the address
that called it, but it is agnostic to what that address is. *The account details are
abstracted such that only the address matters*. The address is retrieved using
the `get_caller_address` module from the StarkNet common library.

Perhaps an address comes from an account contract that does not check signatures. That
is valid. The user of such an account would need to consider whether that is a good idea - for anyone could use their account. From the network perspective, as long as an
operator is paid a fee to compensate their efforts, they do not require a
specific signature to have been used and checked.

## Application deployment

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
        syscall_ptr : felt*
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        number_to_store : felt
    ):
    # Fetch the address of the contract that called this function.
    let (account_address) = get_caller_address()
    balance.write(number_to_store)
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
        stored_number : felt
    ):
    let (stored_number) = balance.read()
    return (stored_number)
end
```

This could be deployed to StarkNet. For this example,
the pytest framework will be used to highlight the mechanics.
The full test code will follow, but the deployment of the app
wil take the following form:
```
application = await deploy(starknet, "contracts/Application.cairo")
```

## Account design

This account will have the following features:

- A single owner
- Verification of ownership using a `private_key` and `public_key` pair.
- The ECDSA Signature algorithm using the native StarkNet curve.

The full code for the account is from the MIT licensed OpenZeppelin repository and
can be found
[here](https://github.com/OpenZeppelin/cairo-contracts/blob/main/contracts/Account.cairo).

A snippet showing the main function from the `Account.cairo` code is shown
below. The contract:

- Accepts `sig_r` and `sig_s` values that the users wallet produces during signing.
- Checks the those signatures match the:
    - Stored L2 public key for the user (Not seen below, but stored in the contract).
    - The `calldata` body of the transaction (containing the `stored_number` for
    the application)
    - Current `nonce`, ensuring the transaction is only sent once.

```
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
    ) -> (response : felt):
    alloc_locals

    let (__fp__, _) = get_fp_and_pc()
    local message: Message = Message(to, selector, calldata, calldata_size=calldata_len, nonce)
    local signed_message: SignedMessage = SignedMessage(&message, sig_r, sig_s)

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

```
The account contract will be deployed with in the test framework:
```
account = await deploy(starknet, "contracts/Account.cairo")
```
First the account contract is deployed. This would require funds to deploy and
so may be deployed by another account, or by a special bridge from L1 with
an account-deploying capability.

## Account initialisation

The account expects to be initialized with a layer 2 public key.

The public key may be generated by a wallet as follows:

- Have a mnemonic seed (may be already used for other networks e.g., Ethereum)
- Have the ability to generate a keypair on the StarkNet-specific curve (ECDSA)
- Use the Hierarchical Deterministic path for StarkNet to produce unique keys
that do not overlap with keys used in other networks. Likely using [EIP-2645](https://eips.ethereum.org/EIPS/eip-2645).

For this example, no specific production wallet is used, and instead StarkWare
python module that comes with cairo-lang installation is used for signature operations.
A fake private key is used. A made up L1 Ethereum account will also be associated
with the account for the emergency L2 failure fallback mode.

In the test function, this will take the form:
```sh
from utils.Signer import Signer

L1_ADDRESS = pretend_L1_address_goes_here
signer = Signer(fake_private_key_goes_here)

# Interact with the deployed account contract to initialize it.
await account.initialize(signer.public_key, L1_ADDRESS).invoke()
```
The account contract has now been set up with a single enshrined public key
it will expect that every transaction it receives will be signed by that key.
All other transaction will not be valid and cannot be included in StarkNet.

## Finally using the L2 account

The account may now be used.

1. Sign a transaction
2. Call the execute() function of the account

The account will check the signature and execute the transaction.

```
my_number = 888878
nonce = 1
store_my_number = signer.build_transaction(
    account, application.contract_address,
    'store_number', [my_number], nonce)
await store_my_number.invoke()
```
The `build_transaction` function is calling the `execute` function of the contract.

# Entire test module

Test the application and account model locally using the `pytest` module
that come with the `cairo-lang` installation.
```
- contracts
    - Application.cairo
    - Account.cairo
- test
    - application_test.py
```

Run:
```
pytest test/application_test.py
```

```sh
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet
from utils.Signer import Signer
from utils.deploy import deploy

# Arbitrary fake private key
signer = Signer(5858585858585858)
# ethereum.eth
L1_ADDRESS = 0xde0b295669a9fd93d5f28d9ec85e40f4cb697bae

@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()


@pytest.fixture(scope='module')
async def test_number_application():
    # Create a local StarkNet object for testing.
    starknet = await Starknet.empty()

    # Deploy the user account contract.
    account = await deploy(starknet, "contracts/Account.cairo")
    # Set up the account for the user.
    await account.initialize(signer.public_key, L1_ADDRESS).invoke()

    # Deploy the number-storing application.
    application = await deploy(starknet, "contracts/Application.cairo")

    initialize = signer.build_transaction(
        account, erc20.contract_address, 'initialize', [], 0)
    await initialize.invoke()
    return starknet, erc20, account

```

The
[utils/Signer.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/test/utils/Signer.py)
is visible below:

```
from starkware.crypto.signature.signature import pedersen_hash, private_to_stark_key, sign
from starkware.starknet.public.abi import get_selector_from_name


class Signer():
    def __init__(self, private_key):
        self.private_key = private_key
        self.public_key = private_to_stark_key(private_key)

    def sign(self, message_hash):
        return sign(msg_hash=message_hash, priv_key=self.private_key)

    def build_transaction(self, account, to, selector_name, calldata, nonce):
        selector = get_selector_from_name(selector_name)
        message_hash = hash_message(to, selector, calldata, nonce)
        (sig_r, sig_s) = self.sign(message_hash)
        return account.execute(to, selector, calldata, nonce, sig_r, sig_s)


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
        return pedersen_hash(hash_calldata(calldata[1:]), calldata[0])
```

The
[utils/deploy.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/test/utils/deploy.py)
module is visible below:

```
from starkware.starknet.testing.contract import StarknetContract
from starkware.starknet.compiler.compile import compile_starknet_files


async def deploy(starknet, path):
    contract_definition = compile_starknet_files([path], debug_info=True)
    contract_address = await starknet.deploy(contract_definition)

    return StarknetContract(
        starknet=starknet,
        abi=contract_definition.abi,
        contract_address=contract_address,
    )
```

# Application deployment

The steps to deploy are similar to that of the test suite.

The additional requirement is that the public key for StarkNet
must be produced from the private key using the curve that is
different from the Ethereum curve. This requires wallets to implement
a function like that used in
[signer.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/test/utils/Signer.py), which calls StarkWare's crypto module visibile
[here](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/crypto/starkware/crypto/signature/signature.py).

Account setup:
```
starknet-compile Account.cairo \
    --output Account_compiled.json \
    --abi Account_contract_abi.json

starknet deploy --contract Account_compiled.json \
    --network=alpha

starknet invoke \
    --network=alpha \
    --address ADDRESS_OF_DEPLOYED_ACCOUNT_CONTRACT \
    --abi Account_contract_abi.json \
    --function Initialize \
    --inputs L2_PUBKEY L1_ADDRESS
```
Application setup:
```
starknet-compile Application.cairo \
    --output Application_compiled.json \
    --abi Application_contract_abi.json

starknet deploy --contract Application_compiled.json \
    --network=alpha
```
Interact with application. Note that the `invoke` call is to the
Account contract, not the application contract.

- `APPLICATION_ADDRESS`: From above deployment.
- `FUNCTION_SELECTOR`: starknet_keccak('store_number')) with ASCII encoding.
- `CALLDATA`: An array with the data (E.g., number to store `[888878]`)
- `NONCE`: Whatever the account nonce is up to (can be viewed in Account contract).
- `SIG_R` and `SIG_S`: Wallet signature of the Pedersen hash of the message as follows:

Message hash to sign:
```
hash(
    hash(
        hash(APPLICATION_ADDRESS, FUNCTION_SELECTOR),
        hash(CALLDATA)),
    NONCE)
```

Call invoke the `Execute` function

```
starknet invoke \
    --network=alpha \
    --address ADDRESS_OF_DEPLOYED_ACCOUNT_CONTRACT \
    --abi Account_contract_abi.json \
    --function Execute \
    --inputs
        APPLICATION_ADDRESS \
        FUNCTION_SELECTOR \
        CALLDATA \
        NONCE \
        SIG_R \
        SIG_S
```

If you are interested in this topic, ways to contribute are:

- Review, audit, make PRs or open issues in the OpenZeppelin Cairo
[contracts library](https://github.com/OpenZeppelin/cairo-contracts)
to build a strong set of contracts for the community to benefit from.
- Work with wallet teams to support the StarkNet ECDSA curve outlined
[here](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/crypto/starkware/crypto/signature/signature.py)
- Work to standardize derivation paths as appropriate, perhaps using
[EIP-2645](https://eips.ethereum.org/EIPS/eip-2645).
- Use and contribute to the [Nile](https://github.com/martriay/nile)
test framework for Cairo contract development.
- Learn [Cairo](https://www.cairo-lang.org/docs/) to write efficient native applications for StarkNet.