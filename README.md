# Diamonds - Introduction

![/assets/qrcode.png](/assets/qrcode.png)

## The parts of a diamond

### Diamond

Main contract, no need to create a new address when upgrading!

### Facets

A deployed contract containing functions that can be attached to a diamond

### functionSelectors

A mappping of the function cutted into the diamonds and their Facet

```
getSubscriptionId() => 0x3F382DbD960E3a9bbCeaE22651Ef8158d2791550
AirdropRegisters() => 0x3F382DbD960E3a9bbCeaE22651Ef8158d2791550
AdjustAirdropPeriod() => 0x3F382DbD960E3a9bbCeaE22651Ef8158d2791550
DistributeCommunityShare(uint256,uint256) => 0x0910FcCD3E3a349944415BCEf75c5b48DAeCd617
```

We are not using the string but the function signature
```
// javascript
web3.eth.abi.encodeFunctionSignature('DistributeCommunityShare(uint256,uint256)');

//solidity
msg.sig
```
So it becomes

```
0xde3d9fb7 => 0x3F382DbD960E3a9bbCeaE22651Ef8158d2791550
0xee944ba7 => 0x3F382DbD960E3a9bbCeaE22651Ef8158d2791550
0x3e52cb99 => 0x3F382DbD960E3a9bbCeaE22651Ef8158d2791550
0xd44e9f70 => 0x0910FcCD3E3a349944415BCEf75c5b48DAeCd617
```

### Fallback fn

Lets play in Remix!

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {
    event FallbackCalled(address,bytes,uint);

    fallback() external payable {
        emit FallbackCalled(msg.sender, msg.data, msg.value);
    }
}
```


### Storage: AppStorage vs Diamond Storage

**AppStorage**: the Diamond storage is a struct that begins at the position 0 in the contract storage.

All the variables must be in the same struct

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TestFacet {

    struct AppStorage {
        uint8 firstVar;
        uint8 secondVar;
        uint8 thirdVar;
        uint8 fourthVar;
        string fifthVar;
    }

    AppStorage internal s;

    function myFacetFunction(uint8 n) external {
        s.firstVar = n;
    }
}
```
**DiamondStorage**: instead of having all the variables in a single struct we can have multiple structs. In AppStorage the struct must start from 0, in DiamondStorage each "struct storage" has to define where it starts.

```solidity
library LibA {
  struct DiamondStorage {
    address owner;
    bytes32 dataA;
  }
  function diamondStorage() internal pure returns(DiamondStorage storage ds) {
    bytes32 storagePosition = keccak256("diamond.storage.LibA");
    // or bytes32 storagePosition = keccak256(abi.encodePacked(ERC1155.interfaceId, ERC1155.name, address(this)));
    assembly {ds.slot := storagePosition}
  }
}

// Our facet uses the Diamond Storage defined above.
contract FacetA {

  function setDataA(bytes32 _dataA) external {
    LibA.DiamondStorage storage ds = LibA.diamondStorage();
    require(ds.owner == msg.sender, "Must be owner.");
    ds.dataA = _dataA;
  }

  function getDataA() external view returns (bytes32) {
    return LibA.diamondStorage().dataA;
  }
}
```

https://eips.ethereum.org/EIPS/eip-2535#facets-state-variables-and-diamond-storage

## Basic Diamonds functions

### cut()
The cut functions is used to connect functions to facets.

### loupe()
The function used to get all the functionSelectors

### fallback()

When calling the Diamond, instead read from the functionSelectors where is the called function (which facet) and do a delegate call to the facet

## Play with Nick's code!

Clone the repo
```
git clone git@github.com:mudgen/diamond-2-hardhat.git
```

Install the project
```
cd diamond-2-hardhat
npm install
```

## A framework for Diamonds: gemcutter

### Install

Clone the repo
```
git clone git@github.com:marcocastignoli/habitat_poc.git
```

Install (it takes a while because of Sourcify)
```
cd habitat_poc/
yarn
```

### While it's installing read this

* developed for 0xhabitat
* the framework simplifies working with diamonds both locally and remotly
* CLI tool to easily cut facets and deploy/upgrade
* Check the docs (they don't represent the actual implementation https://0xhabitat.org/docs/Developers/Gemcutter)

### Run the development environment

```
yarn dev:start
```

### Configure

```
yarn diamond:init:test
```

### Run the tests

```
yarn test
```