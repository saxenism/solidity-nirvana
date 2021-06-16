# What are enums in Solidity?

*enums* or members of enumerated lists, in Solidity work much like enums in any other language. For me, it is basically a tool to reduce the stress on my *mental RAM* while writing code or smart contracts.

## Typical Use Case:

Suppose you want to give your user an option to choose his favorite squad among 9 available options. The available options would be:
* Black Bull
* Silver Eagle
* Blue Rose
* Golden Dawn
* Green Mantis
* Crimson Lion
* Aqua Deer
* Purple Orca
* Coral Peacock

Suppose you intend to use the variable `favSquad` to denote your user's favorite *Black Clover* squad and use that variable in subsequent logic. 

One option (particularly messy one) would be to allow users to enter `strings` that denote their favorite squads, but that would bring forth a whole new hell of matching proper cases (upper cases and lower cases), also someone could enter
a `string` that is out of scope, like `Spade Kingdom` for example.

Another option would be to assign numbers to each squad (0-8) based on their index, most probably like a mapping. That would work, but you'll have to always remember which number represents
which squad while coding, which would be pretty uncomfortable.

Hence, we pivot towards using enums. Enums restrict a variable to have one of only a few predefined values. The values in this enumerated list are called enums.
With the use of enums it is possible to reduce the number of bugs in your code. 
This helps you to not make a mistake, to enter something out of the domain, while entering data and also **improves the program readability**.

For example, this is how we would use enums: 
```solidity
pragma solidity ^ 0.8.0;

contract testEnums {
  enum CloverSquad {
    BlackBull,
    GoldenDawn,
    SilverEagle,
    BlueRose,
    CrimsonLion,
    GreenMantis,
    CoralPeacock,
    PurpleOrca,
    AquaDeer
  }
  
  CloverSquad favSquad;
  CloverSquad firstRankedSquad = CloverSquad.GoldenDawn;
  
  function getFirstRankedSquad() public view returns (CloverSquad) {
    return firstRandkedSquad;
  }
}
```

## Taking enum inputs and checking invalid inputs

A question that might arise in your minds (it did in mine too 😅) is how do we take enums as input from the user of our smart contract. We do this by typecasting the enums and checking
if it is out of range or not.

Example:
```solidity

// Yes, you can use your enums in mappings.
mapping (address => CloverSquad) public playerSquad;

function selectFavoriteSquad(uint userFavSquad) external {
  require(userFavSquad <= uint(CloverSquads.AquaDeer), "Choose from 0 to 8");
  playerSquad[msg.sender] = CloverSquad(userFavSquad);
  // Further Logic....
}
```

## Further Reading

That is about everything that you will need to know about *enums* to be well on your way, but if you really want to get deeper into *enums*, I would suggest reading [this incredibly detailed article](https://gist.github.com/cameel/4d739dc452645539c431ace0d326f146) on *enums*.



**Thank you & Godspeed.**
