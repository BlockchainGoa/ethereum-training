# truffle-tutorial
Using [PetShop](http://truffleframework.com/tutorials/pet-shop) to demonstrate truffle

## Requirements
* [Node.js](https://nodejs.org/en/) LTS & npm
* [Git](https://git-scm.com/)
* [TestRPC](https://github.com/ethereumjs/testrpc)
* [Truffle](http://truffleframework.com/)

## Steps

1. Create truffle project structure
```console
truffle init bare
```

2. Create `Adoption.sol` in `/contracts` and paste the code below
```javascript
pragma solidity ^0.4.15;

contract Adoption {

  address[16] public adopters;

  function adopt(uint8 petId) public returns (uint8) {
    require(petId >= 0 && petId <= 15);
    adopters[petId] = msg.sender;
    return petId;
  }

  function getAdopters() public returns (address[16]) {
    return adopters;
  }
}
```

3. Compile contract
```console
truffle compile
```
Will create a `/build` directory and write output to it

4. Create a migration file `2_adoption_migration.js` in `/migrations`
```javascript
var Adoption = artifacts.require("./Adoption.sol");

module.exports = function(deployer) {
  deployer.deploy(Adoption);
};
```

5. Run testrpc
```console
testrpc --port 8545
```

6. Migrate contract
```console
truffle migrate
```

7. Create file for test cases `TestAdoption.sol` in `/test`
```javascript
pragma solidity ^0.4.15;

import "truffle/Assert.sol";
import "truffle/DeployedAddresses.sol";
import "../contracts/Adoption.sol";

contract TestAdoption {
  Adoption adoption = Adoption(DeployedAddresses.Adoption());

  function testUserCanAdoptPet() {
    uint returnedId = adoption.adopt(8);

    uint expected = 8;

    Assert.equal(returnedId, expected, "Adoption of pet ID 8 should be recorded.");
  }

  function testGetAdopterAddressByPetId() {
    address expected = this;

    address adopter = adoption.adopters(8);

    Assert.equal(adopter, expected, "Owner of pet ID 8 should be recorded.");
  }

  function testGetAdopterAddressByPetIdInArray() {
    address expected = this;

    address[16] memory adopters = adoption.getAdopters();

    Assert.equal(adopters[8], expected, "Owner of pet ID 8 should be recorded.");
  }

}
```

8. Run tests
```console
truffle test
```

---

## JS example

For test cases in JS, and writing tests for payable functions, refer to [conference-truffle](https://github.com/aarongoa/conference-truffle)