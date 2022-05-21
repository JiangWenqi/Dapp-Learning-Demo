# [Pet Shop](https://trufflesuite.com/tutorial/)

## Contents

1. [Setting up the development environment](#1-setting-up-the-development-environment)
2. [Creating a Truffle project using a Truffle Box](#2-creating-a-truffle-project-using-a-truffle-box)
3. [Writing the smart contract](#3-writing-the-smart-contract)
4. [Compiling and migrating the smart contract](#4-compiling-and-migrating-the-smart-contract)
5. [Testing the smart contract](#5-testing-the-smart-contract)
6. [Creating a user interface to interact with the smart contract](#6-creating-a-user-interface-to-interact-with-the-smart-contract)
7. Interacting with the dapp in a browser

---

## 1. Setting up the development environment

```bash
brew install git
brew install node
brew install --cask ganache

npm install -g truffle
```

## 2. Creating a Truffle project using a Truffle Box

```bash
mkdir 02-truffle-pet_shop
cd 02-truffle-pet_shop

truffle unbox pet-shop
```

### Directory structure

The default Truffle directory structure contains the following:

- `contracts/`: Contains the Solidity source files for our smart contracts. There is an important contract in here called `Migrations.sol`, which we'll talk about later.
- `migrations/`: Truffle uses a migration system to handle smart contract deployments. A migration is an additional special smart contract that keeps track of changes.
- `test/`: Contains both JavaScript and Solidity tests for our smart contracts
- `truffle-config.js`: Truffle configuration file

The pet-shop Truffle Box has extra files and folders in it, but we won't worry about those just yet.

## 3. Writing the smart contract

### `contracts/Adoption.sol`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.5.0;

contract Adoption {
    address[16] public adopters;

    // Adopting a pet
    function adopt(uint256 petId) public returns (uint256) {
        require(petId >= 0 && petId <= 15);

        adopters[petId] = msg.sender;

        return petId;
    }

    // Retrieving the adopters
    function getAdopters() public view returns (address[16] memory) {
        return adopters;
    }
}
```

#### Things to notice:

1. The minimum version of Solidity required is noted at the top of the contract: `pragma solidity ^0.8.0;`. The `pragma` command means "additional information that only the compiler cares about", while the caret symbol (`^`) means "the version indicated or higher".
2. Like JavaScript or PHP, statements are terminated with semicolons.
3. The address of the person or smart contract who called this function is denoted by `msg.sender`.
4. `memory` gives the data location for the variable.
5. [view functions](https://solidity.readthedocs.io/en/latest/contracts.html#view-functions)

## 4. Compiling and migrating the smart contract

### Compilation

Solidity is a compiled language, meaning we need to compile our Solidity to bytecode for the Ethereum Virtual Machine (EVM) to execute. Think of it as translating our human-readable Solidity into something the EVM understands.

```bash
truffle compile
```

### Migration

Now that we've successfully compiled our contracts, it's time to migrate them to the blockchain!

> Before we can migrate our contract to the blockchain, we need to have a blockchain running. For this tutorial, we're going to use `Ganache`, a personal blockchain for Ethereum development you can use to deploy contracts, develop applications, and run tests. If you haven't already, download `Ganache` and double click the icon to launch the application. This will generate a blockchain running locally on port `7545`.

![Ganache](https://trufflesuite.com/img/tutorials/pet-shop/ganache-initial.png)

1. Create a new file named `2_deploy_contracts.js` in the `migrations/ directory`.

2. Add the following content to the `2_deploy_contracts.js` file:

   ```bash
   var Adoption = artifacts.require("Adoption");

   module.exports = function (deployer) {
   deployer.deploy(Adoption);
   };
   ```

3. Migrate the contract to the blockchain. `truffle migrate`

   ```text
   Compiling your contracts...
   ===========================
   > Everything is up to date, there is nothing to compile.


   Starting migrations...
   ======================
   > Network name:    'development'
   > Network id:      5777
   > Block gas limit: 6721975 (0x6691b7)


   1_initial_migration.js
   ======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x4bf716aa5f63d3ca27d0797ec7dcaeed8790cbe1a255fb21b995b187af517a98
   > Blocks: 0            Seconds: 0
   > contract address:    0x774C58dF344a1393ECB51E62cd4Ac804a0891070
   > block number:        1
   > block timestamp:     1653168740
   > account:             0x1A80aE7Fa79d27f4A7C500Dbcd23925c90391E5A
   > balance:             99.99616114
   > gas used:            191943 (0x2edc7)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00383886 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00383886 ETH


   2_deploy_contracts.js
   =====================

   Deploying 'Adoption'
   --------------------
   > transaction hash:    0x47a437f650ab9f5b4e85bd74d28fd1787fa888ac49f0ac74e2bfa2b0cc438521
   > Blocks: 0            Seconds: 0
   > contract address:    0x788e43521cC83936FddD2b64cAa135961d42764b
   > block number:        3
   > block timestamp:     1653168740
   > account:             0x1A80aE7Fa79d27f4A7C500Dbcd23925c90391E5A
   > balance:             99.99123808
   > gas used:            203815 (0x31c27)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.0040763 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:           0.0040763 ETH

   Summary
   =======
   > Total deployments:   2
   > Final cost:          0.00791516 ETH

   ```

In `Ganache`, note that the state of the blockchain has changed. The blockchain now shows that the `current block`, previously `0`, is now `4`. In addition, while the first account originally had `100 ether`, it is now lower, due to the transaction costs of migration. We'll talk more about transaction costs later.
![Ganache after migration](https://trufflesuite.com/img/tutorials/pet-shop/ganache-migrated.png)

You've now written your first smart contract and deployed it to a locally running blockchain. It's time to interact with our smart contract now to make sure it does what we want.

## 5. Testing the smart contract

### Testing the smart contract using Solidity

Truffle is very flexible when it comes to smart contract testing, in that tests can be written either in `JavaScript` or `Solidity`. In this tutorial, we'll be writing our tests in Solidity.

1. Create a new file named `TestAdoption.sol` in the `test/` directory.

2. Add the following content to the `TestAdoption.sol` file:

   ```solidity
   // SPDX-License-Identifier: UNLICENSED
   pragma solidity ^0.5.0;

   import "truffle/Assert.sol";
   import "truffle/DeployedAddresses.sol";
   import "../contracts/Adoption.sol";

   contract TestAdoption {
       // The address of the adoption contract to be tested
       Adoption adoption = Adoption(DeployedAddresses.Adoption());

       // The id of the pet that will be used for testing
       uint256 expectedPetId = 8;

       // The expected owner of adopted pet is this contract
       address expectedAdopter = address(this);

       // Testing the adopt() function
       function testUserCanAdoptPet() public {
           uint256 returnedId = adoption.adopt(expectedPetId);

           Assert.equal(
               returnedId,
               expectedPetId,
               "Adoption of the expected pet should match what is returned."
           );
       }

       // Testing retrieval of a single pet's owner
       function testGetAdopterAddressByPetId() public {
           address adopter = adoption.adopters(expectedPetId);

           Assert.equal(
               adopter,
               expectedAdopter,
               "Owner of the expected pet should be this contract"
           );
       }

       // Testing retrieval of all pet owners
       function testGetAdopterAddressByPetIdInArray() public {
           // Store adopters in memory rather than contract's storage
           address[16] memory adopters = adoption.getAdopters();

           Assert.equal(
               adopters[expectedPetId],
               expectedAdopter,
               "Owner of the expected pet should be this contract"
           );
       }
   }
   ```

- The first two imports are referring to global Truffle files, not a `truffle` directory. You should not see a `truffle` directory inside your `test/` directory.

- Then we define three contract-wide variables:
  - First, one containing the smart contract to be tested, calling the `DeployedAddresses` smart contract to get its address.
  - Second, the `id` of the pet that will be used to test the adoption functions.
  - Third, since the `TestAdoption` contract will be sending the transaction, we set the expected adopter address to this, a contract-wide variable that gets the current contract's address.

### Running the tests: `truffle test`

```text
Using network 'development'.


Compiling your contracts...
===========================
> Compiling ./test/TestAdoption.sol
> Artifacts written to /var/folders/30/zdxbr0313yb8s6jz01__yfb80000gn/T/test--28371-qIQe7IE3nhlQ
> Compiled successfully using:
   - solc: 0.5.16+commit.9c3226ce.Emscripten.clang


  TestAdoption
    ✔ testUserCanAdoptPet (202ms)
    ✔ testGetAdopterAddressByPetId (177ms)
    ✔ testGetAdopterAddressByPetIdInArray (212ms)


  3 passing (8s)
```

## 6. Creating a user interface to interact with the smart contract
