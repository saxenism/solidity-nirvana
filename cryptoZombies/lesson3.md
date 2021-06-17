# [Lesson 3](https://share.cryptozombies.io/en/lesson/3/share/saxenismgmail.com):
1. After you deploy a contract to Ethereum, it is immutable. It can never be modified/updated again.
For this reason, if often makes sense to have functions that will allow you to update key portions of your dApp
2. Ownable contract: Owners(contract creators) have special priviliges. It has the following three functions:
    a. When a contract is deployed, its constructor sets the owner to msg.sender (the person who deployed it)
    b. It adds an onlyOwner modifier, which can restirct access to certain functions to only the owner
    c. It allows you to transfer the contract to a new owner
3. Once you inherit from the Ownable contract, you can use the onlyOwner function modifier. This ensures that the function caller is indeed the contract owner or not
4. In Solidity, your users have to pay every time they execute a function on your DApp using a currency called gas. So, basically, users have to spend ETH in order to execute
    functions on your DApp.
5. How much gas is required to execute a function depends on how complex that function's logic is. Each individual operation has a gas cost based roughly on how much 
computing resources will be required to perform that operation. The total gas cost of your function is the sum of the gas costs of all its individual operations.
Therefore, code optimization is much much more important in Ethereum than in other programming languages.
Because, if your code is slopp, then your users are going to pay a premium to execute your functions -- and this could add up to millions of dollars in unnecessary fees across thousand of users.
6. Choosing either of uint8, uint16, uint32, uint256 will result in the same gas fee because Ethereum reserves the same space for each, irrespective of what uint you choose. But you can save on 
    costs when working with multiple uints inside of a struct. Also, for this to happen, you would want to cluster identical data types together (ie put them next to each other in the struct)
7. Solidity provides some native units for dealing with time.
    The variable *now* will return the current unix timestamp of the latest block (the number of seconds that have passed since January 1st 1970).
    Solidity also contains the time units seconds, minutes, hours, days, weeks and years. 
8.  We can pass a storage pointer to a struct as an argument to a private or internal function.
9. An important security practice is to examine all your public and external functions, and try to think of ways users might abuse them. Because, unless these functions have a modifier like onlyOwner,
    any user can call them and pass them any data they want to.
10. The custom function modifier (like onlyOwner) can also take some parameters. The following example will clear things up:
    ```solidity
    mapping (uint => uint) public age;
    // Modifier that requires this user to be older than a certain age:
    modifier olderThan(uint _age, uint _userId) {
    require(age[_userId] >= _age);
    _;
    }
    function driveCar(uint _userId) public olderThan(16, _userId) {
    // Some function logic
    }
    ```
11. Remember how we used memory pointer type along with string in function parameters. Similar to memory we have *calldata* but it's only available to external functions
12. Since view functions only needs to query your local Ethereum node to run the function, it doesn't actually have to create a transaction on the blockchain, which would need to run
    on every single node, and cost gas. Therefore, view functions don't cost any gas when they're called externally by a user.
    Optimize your DApp's gas usage for your users by using read-only external view functions wherever possible.
    If a view function is called internally from another function in the same contract that is not a view function, it will still cost gas. This is because the other function 
    creates a transaction on Ethereum, and will still need to be verified from every node. So view functions are only free when they're called externally.
13. One of the more expensive operations in Solidity is using storage — particularly writes.
    This is because every time you write or change a piece of data, it’s written permanently to the blockchain. Forever! Thousands of nodes across the world need to store that data on
    their hard drives, and this amount of data keeps growing over time as the blockchain grows. So there's a cost to doing that.
    In order to keep costs down, you want to avoid writing data to storage except when absolutely necessary. Sometimes this involves seemingly inefficient programming logic — like
    rebuilding an array in memory every time a function is called instead of simply saving that array in a variable for quick lookups.
    In most programming languages, looping over large data sets is expensive. But in Solidity, this is way cheaper than using storage if it's in an external view function, since 
    view functions don't cost your users any gas. (And gas costs your users real money!).
    An example illustrating how to declare arrays in memory:
    ```solidity
    function getArray() external pure returns(uint[] memory) {
        // Instantiate a new array in memory with a length of 3
        uint[] memory values = new uint[](3);
        // Put some values to it
        values[0] = 1;
        values[1] = 2;
        values[2] = 3;
        return values;
    }
    ```
14. for loops will be preferred over mapping solutions, if it can save gas cost. 

<br/>
<br/>

# Solidity Code from Lesson 3:

##ownable.sol
```solidity
pragma solidity >=0.5.0 <0.6.0;

/**
* @title Ownable
* @dev The Ownable contract has an owner address, and provides basic authorization control
* functions, this simplifies the implementation of "user permissions".
*/
contract Ownable {
  address private _owner;

  event OwnershipTransferred(
    address indexed previousOwner,
    address indexed newOwner
  );

  /**
  * @dev The Ownable constructor sets the original `owner` of the contract to the sender
  * account.
  */
  constructor() internal {
    _owner = msg.sender;
    emit OwnershipTransferred(address(0), _owner);
  }

  /**
  * @return the address of the owner.
  */
  function owner() public view returns(address) {
    return _owner;
  }

  /**
  * @dev Throws if called by any account other than the owner.
  */
  modifier onlyOwner() {
    require(isOwner());
    _;
  }

  /**
  * @return true if `msg.sender` is the owner of the contract.
  */
  function isOwner() public view returns(bool) {
    return msg.sender == _owner;
  }

  /**
  * @dev Allows the current owner to relinquish control of the contract.
  * @notice Renouncing to ownership will leave the contract without an owner.
  * It will not be possible to call the functions with the `onlyOwner`
  * modifier anymore.
  */
  function renounceOwnership() public onlyOwner {
    emit OwnershipTransferred(_owner, address(0));
    _owner = address(0);
  }

  /**
  * @dev Allows the current owner to transfer control of the contract to a newOwner.
  * @param newOwner The address to transfer ownership to.
  */
  function transferOwnership(address newOwner) public onlyOwner {
    _transferOwnership(newOwner);
  }

  /**
  * @dev Transfers control of the contract to a newOwner.
  * @param newOwner The address to transfer ownership to.
  */
  function _transferOwnership(address newOwner) internal {
    require(newOwner != address(0));
    emit OwnershipTransferred(_owner, newOwner);
    _owner = newOwner;
  }
}
```

## zombiefactory.sol
```solidity
pragma solidity >=0.5.0 <0.6.0;

import "./ownable.sol";

contract ZombieFactory is Ownable {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;
    uint cooldownTime = 1 days;

    struct Zombie {
      string name;
      uint dna;
      uint32 level;
      uint32 readyTime;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string memory _name, uint _dna) internal {
        uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime))) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        emit NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        randDna = randDna - randDna % 100;
        _createZombie(_name, randDna);
    }

}

```

## zombiefeeding.sol
```solidity
pragma solidity >=0.5.0 <0.6.0;

import "./zombiefactory.sol";

contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}

contract ZombieFeeding is ZombieFactory {

  KittyInterface kittyContract;

  function setKittyContractAddress(address _address) external onlyOwner {
    kittyContract = KittyInterface(_address);
  }

  function _triggerCooldown(Zombie storage _zombie) internal {
    _zombie.readyTime = uint32(now + cooldownTime);
  }

  function _isReady(Zombie storage _zombie) internal view returns (bool) {
      return (_zombie.readyTime <= now);
  }

  function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) internal {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    require(_isReady(myZombie));
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
    _triggerCooldown(myZombie);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }
}
```

## zombiehelper.sol
```solidity
pragma solidity >=0.5.0 <0.6.0;

import "./zombiefeeding.sol";

contract ZombieHelper is ZombieFeeding {

  // 1. Define levelUpFee here

  modifier aboveLevel(uint _level, uint _zombieId) {
    require(zombies[_zombieId].level >= _level);
    _;
  }

  // 2. Insert levelUp function here

  function changeName(uint _zombieId, string calldata _newName) external aboveLevel(2, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].name = _newName;
  }

  function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].dna = _newDna;
  }

  function getZombiesByOwner(address _owner) external view returns(uint[] memory) {
    uint[] memory result = new uint[](ownerZombieCount[_owner]);
    uint counter = 0;
    for (uint i = 0; i < zombies.length; i++) {
      if (zombieToOwner[i] == _owner) {
        result[counter] = i;
        counter++;
      }
    }
    return result;
  }

}
```


