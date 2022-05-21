# [Pet Shop](https://trufflesuite.com/tutorial/)

## Contents
1. [Setting up the development environment](#1-setting-up-the-development-environment)
2. [Creating a Truffle project using a Truffle Box](#2-creating-a-truffle-project-using-a-truffle-box
3. Writing the smart contract
4. Compiling and migrating the smart contract
5. Testing the smart contract
6. Creating a user interface to interact with the smart contract
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

