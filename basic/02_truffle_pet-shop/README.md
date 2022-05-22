# [Pet Shop](https://trufflesuite.com/tutorial/)

## Contents

1. [Setting up the development environment](#1-setting-up-the-development-environment)
2. [Creating a Truffle project using a Truffle Box](#2-creating-a-truffle-project-using-a-truffle-box)
3. [Writing the smart contract](#3-writing-the-smart-contract)
4. [Compiling and migrating the smart contract](#4-compiling-and-migrating-the-smart-contract)
5. [Testing the smart contract](#5-testing-the-smart-contract)
6. [Creating a user interface to interact with the smart contract](#6-creating-a-user-interface-to-interact-with-the-smart-contract)
7. [Interacting with the dapp in a browser](#7-interacting-with-the-dapp-in-a-browser)

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

### Instantiating `web3`

Open `basic/02_truffle_pet-shop/src/js/app.js`, and remove the multi-line comment from within `initWeb3` and replace it with the following:

```javascript
initWeb3: async function () {
    // Modern dapp browsers...
    if (window.ethereum) {
      App.web3Provider = window.ethereum;
      try {
        // Request account access
        await window.ethereum.request({ method: "eth_requestAccounts" });
      } catch (error) {
        // User denied account access...
        console.error("User denied account access");
      }
    }
    // Legacy dapp browsers...
    else if (window.web3) {
      App.web3Provider = window.web3.currentProvider;
    }
    // If no injected web3 instance is detected, fall back to Ganache
    else {
      App.web3Provider = new Web3.providers.HttpProvider(
        "http://localhost:7545"
      );
    }
    web3 = new Web3(App.web3Provider);

    return App.initContract();
},
```

- First, we check if we are using modern dapp browsers or the more recent versions of `MetaMask` where an `ethereum` provider is injected into the window object. If so, we use it to create our web3 object, but we also need to explicitly request access to the accounts with `ethereum.enable()`.

- If the `ethereum` object does not exist, we then check for an injected web3 instance. If it exists, this indicates that we are using an older dapp browser. If so, we get its provider and use it to create our web3 object.

- If no injected web3 instance is present, we create our web3 object based on our local provider. (This fallback is fine for development environments, but insecure and not suitable for production.)

### Instantiating the `contract`

Open `basic/02_truffle_pet-shop/src/js/app.js`, and remove the multi-line comment from within `initContract` and replace it with the following:

```javascript
initContract: function () {
    $.getJSON("Adoption.json", function (data) {
        // Get the necessary contract artifact file and instantiate it with @truffle/contract
        var AdoptionArtifact = data;
        App.contracts.Adoption = TruffleContract(AdoptionArtifact);

        // Set the provider for our contract
        App.contracts.Adoption.setProvider(App.web3Provider);

        // Use our contract to retrieve and mark the adopted pets
        return App.markAdopted();
    });

    return App.bindEvents();
},
```

- We first retrieve the artifact file for our smart contract. **Artifacts are information about our contract such as its deployed address and Application Binary Interface (ABI). The ABI is a JavaScript object defining how to interact with the contract including its variables, functions and their parameters.**

- Once we have the artifacts in our callback, we pass them to `TruffleContract()`. This creates an instance of the contract we can interact with.

- With our contract instantiated, we set its web3 provider using the `App.web3Provider` value we stored earlier when setting up web3.

- We then call the app's `markAdopted()` function in case any pets are already adopted from a previous visit. We've encapsulated this in a separate function since we'll need to update the UI any time we make a change to the smart contract's data.

### Getting The Adopted Pets and Updating The UI

Open `basic/02_truffle_pet-shop/src/js/app.js`, and remove the multi-line comment from within `markAdopted` and replace it with the following:

```javascript
markAdopted: function () {
    var adoptionInstance;

    App.contracts.Adoption.deployed()
        .then(function (instance) {
            adoptionInstance = instance;
            return adoptionInstance.getAdopters.call();
        })
        .then(function (adopters) {
            for (i = 0; i < adopters.length; i++) {
                if (adopters[i] !== "0x0000000000000000000000000000000000000000") {
                $(".panel-pet")
                    .eq(i)
                    .find("button")
                    .text("Success")
                    .attr("disabled", true);
                }
            }
        })
        .catch(function (err) {
            console.log(err.message);
        });
},
```

- We access the deployed `Adoption` contract, then call `getAdopters()` on that instance.

- We first declare the variable `adoptionInstance` outside of the smart contract calls so we can access the instance after initially retrieving it.

- Using `call()` allows us to read data from the blockchain without having to send a full transaction, meaning we won't have to spend any ether.

- After calling `getAdopters()`, we then loop through all of them, checking to see if an address is stored for each pet. Since the array contains address types, Ethereum initializes the array with 16 empty addresses. This is why we check for an empty address string rather than null or other falsey value.

- Once a `petId` with a corresponding address is found, we disable its adopt button and change the button text to "Success", so the user gets some feedback.

- Any errors are logged to the console.

### Handling the `adopt()` Function

Open `basic/02_truffle_pet-shop/src/js/app.js`, and remove the multi-line comment from within `handleAdopt` and replace it with the following:

```javascript
handleAdopt: function (event) {
    event.preventDefault();

    var petId = parseInt($(event.target).data("id"));

    var adoptionInstance;

    web3.eth.getAccounts(function (error, accounts) {
        if (error) {
            console.log(error);
        }

        var account = accounts[0];

        App.contracts.Adoption.deployed()
            .then(function (instance) {
                adoptionInstance = instance;

                // Execute adopt as a transaction by sending account
                return adoptionInstance.adopt(petId, { from: account });
            })
            .then(function (result) {
                return App.markAdopted();
            })
            .catch(function (err) {
                console.log(err.message);
            });
    });
},
```
- We use web3 to get the user's accounts. In the callback after an error check, we then select the first account.

- From there, we get the deployed contract as we did above and store the instance in `adoptionInstance`. This time though, we're going to send a **transaction** instead of a call. Transactions require a "from" address and have an associated cost. This cost, paid in ether, is called **gas**. The gas cost is the fee for performing computation and/or storing data in a smart contract. We send the transaction by executing the `adopt()` function with both the pet's ID and an object containing the account address, which we stored earlier in `account`.

- The result of sending a transaction is the transaction object. If there are no errors, we proceed to call our `markAdopted()` function to sync the UI with our newly stored data.

## 7. Interacting with the dapp in a browser
### MetaMask
1. Click Import Wallet. In the box marked Wallet Seed, enter the `mnemonic` that is displayed in Ganache.

2. We need to connect `MetaMask` to the blockchain created by `Ganache`. Click the menu that shows "**Main Network**" and select Custom RPC.

3. In the box titled "New Network" enter `http://127.0.0.1:7545`, in the box titled "Chain ID" enter `1337` (Default Chain ID for `Ganache`) and click Save.

4. Each account created by `Ganache` is given 100 ether. You'll notice it's slightly less on the first account because some gas was used when the contract itself was deployed and when the tests were run.

### Installing and configuring lite-server
We can now start a local web server and use the dapp. We're using the lite-server library to serve our static files. This shipped with the pet-shop Truffle Box, but let's take a look at how it works.

1. Open `bs-config.json` in a text editor (in the project's root directory) and examine the contents:

    ```json
    {
    "server": {
        "baseDir": ["./src", "./build/contracts"]
    }
    }
    ```

2. Check `package.json`:
    ```json
         "scripts": {
            "dev": "lite-server",
            "test": "echo \"Error: no test specified\" && exit 1"
        },
    ```

### Using the dapp
Start the local web server:
```bash
npm run dev
```
![pet-shop](https://trufflesuite.com/img/tutorials/pet-shop/dapp.png)

Have fun!


## Reference

https://trufflesuite.com/tutorial/