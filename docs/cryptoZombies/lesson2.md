# [Lesson 2](https://share.cryptozombies.io/en/lesson/2/share/saxenismgmail.com):

1. Addresses:
    The ethereum blockchain is made up of accounts, which you can think of as bank accounts.
    An account has a balance of Ether, and you can send and recieve Ether payments to other accounts, just like your bank account can wire transfer money to other bank accounts
    Each bank account has an address which you can think of like a bank account number.Its a unique identifier that points to an account.
    An address is owned by a specific user or a smart contract.
   Mapping:
    So we can use it as a unique ID for ownership of our zombies. When a user creates new zombies by interacting with our app, we'll set ownership of those zombies
    to the Ethereum address that called the function
    A mapping is essentially a key-value store for storing and looking up data
        mapping(uint => string) userIdToName;
2. msg.sender
    In solidity, there are certain global variables that are available to all functions. One of them is msg.sender
    msg.sender refers to the address of the person (or the smart contract) who called the current function
    In solidity, function execution always needs to start with an external caller. A contract will just sit on the blockchain doing nothing until someone calls one of its functions. So, 
    there will always be a msg.sender
3. require is a keyword in Solidity used for condition checking. If this condition is met, then only a function is executed otherwise, it terminates with an error.
    Example:
    ```solidity
    function sayHiToVitalik (string memory _name) public returns (string memory) {
        require(keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Vitalik")));
        return "Hi Vitalik, thank you for Ethereum!!";
    }
    ```
4. Solidity does not support string comparison natively, so we simply compare the keccak256 hashes of the two strings.
5. Solidity supports inheritence. Hence, instead of writing one big long contract, it makes sense to split your code logic across multiple
    contracts to organize the code.
6.  Really cool and succint inheritence syntax:
    ```solidity
    contract cat is animal {
    } // Here the contract cat inherits from the contract animal :D
    ```
7. Syntax to import one file into another:
    ```solidity
    import './someOtherContract.sol';
    contract newContract is someOtherContract {
    }
    ```
8. In solidity, there are two locations in which you can store variables - in *storage* and in *memory*.
    Storage refers to variables permanently stored on the blockchain.
    Memory variables are temporary, and are erased between external function calls to your contract.
    Think of it like a computer's Hard Disk vs RAM
Example:
 ```solidity
contract SandwichFactory {
    struct Sandwich {
        string name;
        string status;
    }
    Sandwich[] sandwiches;
    function eatSandwich(uint _index) public {
        // Sandwich mySandwich = sandwiches[_index];
        // ^ Seems pretty straightforward, but solidity will give you a warning
        // telling you that you should explicitly declare storage or memory here.
        // So instead, you should declare with the storage keyword, like:
        Sandwich storage mySandwich = sandwiches[_index];
        // ...in which case mySandwich is a pointer to sandwiches[_index]
        // in storage, and...
        mySandwich.status = "Eaten!";
        // ...this will permanently change `sandwiches[_index]` on the blockchain.
        // If you just want a copy, you can use memory:
        Sandwich memory anotherSandwich = sandwiches[_index + 1];
        // ...in which case anotherSandwich will simply be a copy of the 
        // data in memory, and...
        anotherSandwich.status = "Eaten!";
        // ...will just modify the temporary variable and have no effect 
        // on sandwiches[_index + 1]. But you can do this:
        sandwiches[_index + 1] = anotherSandwich;
        // ...if you want to copy the changes back into blockchain storage.
    }
  }
```

9. For a struct People such as:
    ```solidity
    struct People {
        string name;
        uint age;
    }
    People randomMan = People("Sam", "22");
    ```
    You can access the properties of this struct as follows:
    randomMan.age or randomMan.name
10. Types of functions in solidity:
    a. Private
    b. Internal
    c. Public
    d. External
internal is the same as private, except that it's also accessible to contracts that inherit from this contract.
external is the same public, except that these functions can ONLY be called outside the contract.
11. Interacting with other smart contracts on the Ethereum blockchain:
    a. Define an interface (almost the same syntax as that of a contract, but only function signatures are mentioned here.)
    b. Only those function signatures need to be written in the interface, that we actually need to call
    c. Grab the address of the smart contract, you want to call the function from
    d. Initialise the interface you built with this address (much like creating an object)
    e. Example:
    ```solidity
        contract NumberInterface {
            function getNum(address _myAddress) public view returns (uint);
        }
        contract MyContract {
            address NumberInterfaceAddress = 0xabcde122......
            NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
            function someFunction() public {
                uint num = numberContract.getNum(msg.sender);
            }
        }
      ```
 12. In Solidity you can return more than one value from a function :D
 13. This example illustrates, how we manage multiple return values from Solidity function:
 ```solidity
    function multipleReturns() internal returns(uint a, uint b, uint c) {
        return (1, 2, 3);
    }
        function processMultipleReturns() external {
        uint a;
        uint b;
        uint c;
        // This is how you do multiple assignment:
        (a, b, c) = multipleReturns();
        }
        // Or if we only cared about one of the values:
        function getLastReturnValue() external {
        uint c;
        // We can just leave the other fields blank:
        (,,c) = multipleReturns();
    }
```
<br/>
<br/>

# Solidity code from Lesson2

## zombiefactory.sol
```solidity
pragma solidity >= 0.8.4;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits; //(10^16)

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string memory _name, uint _dna) internal {
        zombies.push(Zombie(_name, _dna));
        uint id = zombies.length - 1;
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
pragma solidity ^ 0.8.4;

import "lesson2_zombiefactory.sol";

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

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }
}
```
