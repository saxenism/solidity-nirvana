# [Lesson 1](https://share.cryptozombies.io/en/lesson/1/share/saxenismgmail.com):
1. Solidity code is encapsulated in contracts. A contract is the fundamental building block of Ethereum
applications. All variables and functions belong to a contract and this will be the starting point of all projects
2. State Variables:
    Permanently stored in contract storage (they are written to the Ethereum Blockchain)
    Think of declaring state variables like writing to a DB
3. Arithmetic operations:
    x+y, x-y, x*y, x/y, x%y, x**y(same as x^y)
4. structs
    ```
    struct Person {
        uint age;
        string name;
    }
    ```
5. Arrays {Possible to create an array of structs as well as native data types}
    Fixed arrays -> uint [2] fixedArr
    Dynamic arrays -> uint [] dynamicArr
    If an array is declared as public, other contracts can read from but not write to this array
6. ```
    function createZombies (string memory _name, uint _dna) public {
        underscore is a naming convention to separate the private variables from the global variables(_name) 
    } 
    ```
    Notice the word memory along with the string parameter, it signifies that _name would be memory and not in stack, therefore would not be written to the blockchain, hence cheaper
    The *memory* keyword is required for reference type varialbles: arrays, structs, mapping, strings, etc.
7.  Pushing to the array:
    ```
    Person Satoshi = Person(172, "Satoshi");
    people.push(Satoshi);
    ```
8. In solidity, all functions are public by default. Good practice is to intially make all the functions private and then only make public the functions you want to expost to the world
    Convetion to start private functions with underscore (function _generateZombieID())
9. Return values
    ```
    function sayHello() public returns (string memory) {
    }
    ```
    If a function does not change any values, we can term it as a view function. [function sayHello() public view returns (string memory) {}]
    Pure functions => not accessing any data from the contract.
10. Keccak256 and typecasting:
    Ethereum has the hash function keccak 256 built-in, which is a version of SHA3.
    keccak256 expects a single parameter of type bytes
    This means we have to "pack" any parameter before calling keccak256
11. Compiler supports typecasting
    var = uint8(var) => This shit
12. Events are a way for your contract to communicate to your app front-end that something has happened on the blockchain, which can be listening for certain events and take actions when they happen
    Example declaration:
    ```
    event IntegersAdded(uint x, uint y, uint result); //Include input and ouputs to a function
    function add(uint _x, uint _y) public returns (uint) {
        uint result = _x + _y;
        emit IntegersAdded(_x, _y, result);
        return result;
    }
    ```
    
  
<br/>

# Solidity code from Lesson 1
```solidity
pragma solidity ^ 0.8.0;

contract ZombieWorld {

    event NewZomie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string memory _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
        uint id = zombies.length - 1;
        emit NewZomie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }
}
```
