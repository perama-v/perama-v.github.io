---
layout: page
title:  "Nile"
permalink: /cairo/nile/
toc: false
---

Installation of cairo-lang installs all the the tooling
required for interacting with contracts and StarkNet.

Nile provides a user-friendly solution that makes those operations
more friendly. Here is the quick guide to setting up Nile. Always
defer to the [Nile repository](https://github.com/OpenZeppelin/nile)
for the most up to date information.

Perform these once-only commands to set up a project to work through the
examples.
```sh
mkdir nile-cairo-examples
cd nile-cairo-examples
python3 -m venv env
source env/bin/activate
nile init
```

When writing and testing contracts, there is a few different ways
to develop:

- Writing the contract itself
    - vs-code is a good editor.
    - Install it and run `code .` to open and edit files.
- Compiling the contract with `nile compile`.
- Testing the contract with tests written in python
    - These are located in `/tests`.
    - They are run using `pytest my_test`.
    - This will create a short-lived local starknet for every test.
- Testing by deploying to a local StarkNet
    - Open a new terminal in the project folder and run `nile node`.
    - Then in a different terminal run:
        - `nile deploy contract --alias my_contract`.
    - This will deploy the contract to that local network.
- Testing by deploying to the public testnet
    - In the main project folder run:
        - `nile deploy contract --alias my_contract --network mainnet`.
    - This will deploy the contract to the public network.

When you make a new contract, it is easy to keep the contract name and
the alias the same. E.g. for `vault.cairo`:

```
            # 1           # 2
nile deploy vault --alias vault

# 1 is the name of the contract file used to make the deployment.
# 2 is the name you wish to refer to the contract in future commands.
```

To interact with a contract there are two possibilities:

- Contract functions with `@external`
    - These write to the contract.
    - Use the `nile invoke` command.
- Contract functions with `@view`
    - These retrieve values from the contract.
    - Use the `nile call` command.

```
            # 1   # 2          # 3
nile invoke vault unlock_funds 300

# 1 is the alias previously given to the contract.
# 2 is the function of the contract being used.
# 3 is some value being passed to the function.
```
In this imaginary contract, the vault is unlocking 300 funds.
```
          # 1   # 2
nile call vault view_unlocked_funds

# 1 is the alias previously given to the contract.
# 2 is the function of the contract being used.
```
In the above command, the contract is used in read-only mode to
retrieve information.
