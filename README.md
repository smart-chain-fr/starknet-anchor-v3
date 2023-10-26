# Starknet - anchor

Anchoring smart contracts that supports a SHA-256 message (double felt).

This repository implements an "anchoring" feature via a smart contract on Starknet.
This repository provides smart contracts, tests and scripts for deploying a factory of anchoring contracts.

## prerequisite

scarb - https://docs.swmansion.com/scarb/download.html
```
curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh
```

snforge - https://github.com/foundry-rs/starknet-foundry
```
curl -L https://raw.githubusercontent.com/foundry-rs/starknet-foundry/master/scripts/install.sh | sh
```


## Deployed on Testnet

anchoring contract class hash: `0x6c327a65f1445575597205314eb9b1af7bfb9222f2540f45b013fa2d86870a1`
factory contract class hash: `0x317b17f68ac153f19f04a9b6117b77553910e051ee356ab077417a884c5b4c5`
factory contract instance: `0x4b76da8728b2ad07d5be3d81fe3ffb466a74f272cf94195d79076c75afc0fbe`
an anchoring contract instance has been generated at: `0x0240621f865df3ca79dc1a614d60f06310e19d8c6ac49d337dec6e0246473d2b`


## Smart contracts

This repository involves two contracts: the Factory contract and the Anchoring contract. This repository contains the implementation of factory and anchoring contracts written in cairo v2.3.0.

The Factory contract provides an invokable function (`deploy`) to deploy a new anchor contract on-demand. This function can only be called by the administrator role of the Factory.

The Anchoring contract provides a callable function (`anchor`) to anchor a message. This function can only be called by users that have been whitelisted on the Anchoring contract. The administrator role of the Anchoring contract can `authorize` or `unauthorize` some users.

These two contracts also provides "getter" function that retieves specific information from the storage (more details in the following sections).


### Factory Contract  

The Factory contract is responsible for deploying an Anchoring contract for each client/product.

The creation of an instance of Factory contract (i.e. deployment of a Factory contract) requires two parameters:
- the admin address
- the anchoring contract class hash.

#### Features:
- Administration: The Factory contract contains an admin. Only this admin can use the deployment function of the Anchoring contract.
- Admin Transfer proposal: The admin of the Factory contract can designate another address as a proposed admin.
- Admin change confirmation: The proposed admin of the factory can invoke the `acceptAdmin` entrypoint to become the new admin role, and thereby forfeit the old admin access.

- Deployment: When the admin calls the `deploy` function, it creates a new instance of Anchoring contract. The administrator address of the Anchoring contract must be provided as parameter. Note that the given admin address is also whitelisted at Anchoring contract level.
- Admin Class hash modification: The admin of the Factory contract can specify another Anchoring contract class hash.
- Once an anchoring contract is generated, its owner can be queried (with `get_anchor_contract_owner` entrypoint). 

- The smart contract also provides a reverse map that allow anyone to query the anchoring contracts generated by a specific user address. `get_owner_contract_by_owner_index` entrypoint returns an contract address (for a given user address and an index). For example , `get_owner_contract_by_owner_index(<user_address>, 0)` retrieves the first contract generated by the given `<user_address>`.

- The `get_owner_contract_index` provides the number of contracts generated by a `<user_address>`. For example, the following invocation retrieves the last contract generated by the given user_address: `get_owner_contract_by_index(<user_address>, get_owner_contract_index(<user_address>) - 1)`.



### Anchoring Contract:  

The Anchoring contract is responsible for anchoring a hash (SHA-256). 

#### Features:
- Administration: The Anchoring contract contains an admin role. Only this admin can add/remove a user from the whitelist with `authorize` and `unauthorize` functions .
- Whitelisting: Only the whitelisted address can anchor a message. 

- Anchoring messages: Whitelisted users can anchor a message by calling the `anchor` function (and by specifying a message as parameter).
- Message uniqueness: A message can only be anchored once.
- Timestamping: When a message is anchored, the current block's timestamp is saved and associated to. Notice that this feature is redundant with the timestamp of the transaction.

- Anchor verification: anyone can retrieve anchored messages and their related timestamps without any cost.

- Retrieval: One can retrieve an array of all anchored messages with the `get_anchored_values` function, an array of each of message timestamps with `get_anchored_timestamps` function. Additionally, one can retrieve the timestamp for a specific message that has been anchored with `get_anchored_timestamp` function.



## How to deploy and interact with the contract (using Makefile)

### Setup

- setup your env variables `make setup`. This step needs to be done only once.
- download npm dependencies `make install`. It installs the "node_module" libraries. The "node_modules" directory can be deleted with the `make clean` command.




### Compilation and tests of smart contracts

- Compilation of smart contracts can be launched with command `make compile`. It produces the sierra code (JSON) and the casm code (JSON) in the `target/dev` directory.
Actually the compile command relies on the scarb package manager and in fact calls the `scarb build` command.

- Tests of smart contracts can be launched with command `make test`. It executes all tests
It also relies on he scarb package manager and in fact calls the `scrab test` command. Tests are based on a test framework  Starknet-foundry, so tests can also be launched with command `snforge`.

If you get the following error,  
```
error: Plugin diagnostic: The 'contract' attribute was deprecated, please use `starknet::contract` instead.
 --> lib.cairo:1:1
#[contract]
^*********^
```
you need to remove node_modules with `make clean` because a module contains a deprecated smart contract example. 

This bug Starknet-foundry will be fixed soon. When it is fixed, you must change the dependency in the `Scarb.toml` file.


### Deployment

The repository provides `scripts` directory containing scripts written in TypeScript for deploying smart contracts and interacting with deployed contracts. 

- It relies on the Starknet-js library which can be installed with `make install`.

- Before deploying an instance of a Factory contract , one must declare (only once) the class contract of Anchoring and Factory.
`make declare-anchoring`
It should produce locally a file `deployments/anchoring_class_hash.ts` with the address of the anchoring class hash.
`make declare-factory`
It should produce locally a file `deployments/factory_class_hash.ts` with the address of the factory class hash.


- Once class contract have been declared, an instance can be produced by deploying the factory contract.
`make deploy-factory`
It should produce locally a file `deployments/factory.ts` with the address of the factory smart contract.

- Once the Factory contract has been deployed, the instance is available for producing a new Anchoring contract.
`make generate-anchoring`
It should produce locally a file `deployments/anchoring.ts` with the address of the anchoring smart contract.

### Anchor a hash !
- Anchor a message. The message is a SHA-256 hash (without the '0x' prefix)
`make anchor-message MSG=1c4c85d3c9855a6f87551d7672b5459d8de5844e20c7cdd8526f78885e31eed9`
- you can also specify the Anchoring contract address as a CLI variable
```
make anchor-message MSG=1c4c85d3c9855a6f87551d7672b5459d8de5844e20c7cdd8526f78885e31eed9 ANCHORING=0x240621f865df3ca79dc1a614d60f06310e19d8c6ac49d337dec6e0246473d2b
```
As you can see the message is specified as a 256-bit hash. Behind the `make anchor-message` command there is a script that splits the given hash into 2 felts before calling the `anchor` entrypoint of the Anchoring contract.



    
## How to interact with the deployed contract on `starkscan` explorer
- First of all, go the [contract page into the explorer, into the tab **Read/Write**](https://testnet.starkscan.co/contract/0x44f99149391e619aaaf55c31fbb9fca8cf4ba542ff961a4999305c3e790e17f#read-write-contract), then you should be able to consult the current anchored value by calling `get_anchored_values()` function.
- For the next interactions, you will need to get a wallet that will make you able to interact with functions on the explorer like [ArgentX](https://www.argent.xyz/argent-x/). 
- You will need to create a wallet and fund it with something like `0.001eth` to be fine.
- You can now prepare your message, it can be a number or a string, if you want to do a string you can use a tool like [this](https://string-functions.com/string-hex.aspx) in order to generate an hex version of your string.
-  Call the function `anchor(value)` with value something like `0x<your_hex_converted_value>`.
- Congratulation!









