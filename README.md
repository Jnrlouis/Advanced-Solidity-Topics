# Advanced Solidity Topics

In the Freshman track, we looked at some basic Solidity syntax. We covered variables, data types, functions, loops, conditional flows, and arrays. 

However, Solidity has a few more things, things which will be important through the coding assignments of the Sophomore track and beyond. In this tutorial, we will cover some more important Solidity topics.

## Index
- [Mappings](#mappings)
- [Enums](#enums)
- [Structs](#structs)
- [View and Pure Functions](#view-and-pure-functions)
- [Function Modifiers](#function-modifiers)
- [Events](#events)
- [Constructors](#constructors)
- [Inheritance](#inheritance)
- [Transferring ETH](#transferring-eth)
- [Calling external contracts](#calling-external-contracts)
- [Import statements](#import-statements)
- [Solidity Libraries](#solidity-libraries)

## Mappings
Mappings in Solidity act like hashmaps or dictionaries in other programming languages. They are used to store the data in key-value pairs. 

Mappings are created with the syntax `mapping (keyType => valueType)`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Mapping {
    // Mapping from address to uints
    mapping(address => uint) public myMap;
    
    function get(address _addr) public view returns (uint) {
        // Mapping always returns a value.
        // If the value was never set, it will return the default value.
        // The default value for uint is 0
        return myMap[_addr];
    }

    function set(address _addr, uint _i) public {
        // Update the value at this address
        myMap[_addr] = _i;
    }

    function remove(address _addr) public {
        // Reset the value to the default value.
        delete myMap[_addr];
    }
}
```

We can also create nested mappings, where the `key` points to a second nested mapping. To do this, we set the `valueType` to a mapping itself.

```solidity
contract NestedMappings {
    // Mapping from address => (mapping from uint to bool)
    mapping(address => mapping(uint => bool)) public nestedMap;
    
    function get(address _addr1, uint _i) public view returns (bool) {
        // You can get values from a nested mapping
        // even when it is not initialized
        // The default value for a bool type is false
        return nestedMap[_addr1][_i];
    }

    function set(
        address _addr1,
        uint _i,
        bool _boo
    ) public {
        nestedMap[_addr1][_i] = _boo;
    }

    function remove(address _addr1, uint _i) public {
        delete nestedMap[_addr1][_i];
    }
}
```

## Enums
The word `Enum` stands for `Enumerable`. They are user defined types that contain human readable names for a set of constants, called members. They are commonly used to restrict a variable to only have one of a few predefined values. Since they are just an abstraction for human readable constants, in actuality, they are internally represented as `uint`s.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Enum {
    // Enum representing different possible shipping states
    enum Status {
        Pending,
        Shipped,
        Accepted,
        Rejected,
        Canceled
    }
    
    // Declare a variable of the type Status
    // This can only contain one of the predefined values
    Status public status;
    
    // Since enums are internally represented by uints
    // This function will always return a uint
    // Pending = 0
    // Shipped = 1
    // Accepted = 2
    // Rejected = 3
    // Canceled = 4
    // Value higher than 4 cannot be returned
    function get() public view returns (Status) {
        return status;
    }
    
    // Pass a uint for input to update the value
    function set(Status _status) public {
        status = _status;
    }
    
    // Update value to a specific enum members
    function cancel() public {
        status = Status.Canceled; // Will set status = 4
    }
}
```

## Structs
The concept of structs exists in many high level programming languages. They are used to define your own data types which group together related data. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract TodoList {
    // Declare a struct which groups together two data types
    struct TodoItem {
        string text;
        bool completed;
    }
    
    // Create an array of TodoItem structs
    TodoItem[] public todos;
    
    function createTodo(string memory _text) public {
        // There are multiple ways to initialize structs
        
        // Method 1 - Call it like a function
        todos.push(Todo(_text, false));
        
        // Method 2 - Explicitly set its keys
        todos.push(Todo({ text: _text, completed: false }));
        
        // Method 3 - Initialize an empty struct, then set individual properties
        Todo memory todo;
        todo.text = _text;
        todo.completed = false;
        todos.push(todo);
    }
    
    // Update a struct value
    function update(uint _index, string memory _text) public {
        todos[_index].text = _text;
    }
    
    // Update completed
    function toggleCompleted(uint _index) public {
        todos[_index].completed = !todos[_index].completed;
    }
}
```

## View and Pure Functions
You might have noticed that some of the functions we have been writing specify one of either a `view` or `pure` keyword in the function header. These are special keywords which indicate specific behavior for the function.

Getter functions (those which return values) can be declared either `view` or `pure`. 
- `View`: Functions which do not change any state values
- `Pure`: Functions which do not change any state values, but also don't read any state values

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract ViewAndPure {
    // Declare a state variable
    uint public x = 1;
    
    // Promise not to modify the state (but can read state)
    function addToX(uint y) public view returns (uint) {
        return x + y;
    }
    
    // Promise not to modify or read from state
    function add(uint i, uint j) public pure returns (uint) {
        return i + j;
    }
}
```

## Function Modifiers
Modifiers are code that can be run before and/or after a function call. They are commonly used for restricting access to certain functions, validing input parameters, protecting against certain types of attacks, etc.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Modifiers {
    address public owner;
    
    constructor() {
        // Set the contract deployer as the owner of the contract
        owner = msg.sender;
    }
    
    // Create a modifier that only allows a function to be called by the owner
    modifier onlyOwner() {
        require(msg.sender == owner, "You are not the owner");
        
        // Underscore is a special character used inside modifiers
        // Which tells Solidity to execute the function the modifier is used on
        // at this point
        // Therefore, this modifier will first perform the above check
        // Then run the rest of the code
        _;
    }
    
    // Create a function and apply the onlyOwner modifier on it
    function changeOwner(address _newOwner) public onlyOwner {
        // We will only reach this point if the modifier succeeds with its checks
        // So the caller of this transaction must be the current owner
        owner = _newOwner;
    }
}
```

## Events
Events allow contracts to perform logging on the Ethereum blockchain. Logs for a given contract can be parsed later to perform updates on the frontend interface, for example. They are commonly used to allow frontend interfaces to listen for specific events and update the user interface, or used as a cheap form of storage.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Events {
    // Declare an event which logs an address and a string
    event TestCalled(address sender, string message);
    
    function test() public {
        // Log an event 
        emit TestCalled(msg.sender, "Someone called test()!");
    }
}
```

## Constructors
A `constructor` is an optional function that is executed when the contract is first deployed. You can also pass arguments to constructors.

P.S. - If you remember, we actually used constructors in the Freshman track Cryptocurency and NFT tutorials!

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract X {
    string public name;
    
    // You will need to provide a string argument when deploying the contract
    constructor(string memory _name) {
        // This will be set immediately when the contract is deployed
        name = _name;
    }
}
```

## Inheritance
Inheritance is the procedure by which one contract can inherit the attributes and methods of another contract. Solidity supports multiple inheritance. Contracts can inherit other contract by using the `is` keyword. 

Note: We actually also did Inheritance in the Freshman Track Cryptocurrency and NFT tutorials - where we inherited from the `ERC20` and `ERC721` contracts respectively.

A parent contract which has a function that can be overridden by a child contract must be declared as a `virtual` function.

A child contrant that is going to override a parent function must use the `override` keyword.

The order of inheritance matters if parent contracts share methods or attributes by the same name.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

/* Graph of inheritance
    A
   / \
  B   C
 /   /
D   E

*/

contract A {
    // Declare a virtual function foo() which can be overridden by children
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
}

contract B is A {
    // Override A.foo();
    // But also allow this function to be overridden by further children
    // So we specify both keywords - virtual and override
    function foo() public pure virtual override returns (string memory) {
        return "B";
    }
}

contract C is A {
    // Similar to contract B above
    function foo() public pure virtual override returns (string memory) {
        return "C";
    }
}

// When inheriting from multiple contracts, if a function is defined multiple times, the right-most parent contract's function is used.
contract D is B, C {
    // D.foo() returns "C"
    // since C is the right-most parent with function foo();
    // override (B,C) means we want to override a method that exists in two parents
    function foo() public pure override (B, C) returns (string memory) {
        // super is a special keyword that is used to call functions
        // in the parent contract
        return super.foo();
    }
}

contract E is C, B {
    // E.foo() returns "B"
    // since B is the right-most parent with function foo();
    function foo() public pure override (C, B) returns (string memory) {
        return super.foo();
    }
}

```

## Transferring ETH
There are three ways to transfer ETH from a contract to some other address. However, two of them are no longer recommended methods by Solidity in latest versions, therefore we shall skip those.

Currently, the recommended way to transfer ETH from a contract is to use the `call` function. The `call` function returns a `bool` indicating success or failure of the transfer.

### How to receive Ether in a regular Ethereum account address
If transferring ETH to a regular account (like a Metamask address), you do not need to do anything special as all such accounts can automatically accept ETH transfers.

### How to receive Ether in a contract
However, if you are writing a contract that you want to be able to receive ETH transfers directly, you must have at least one of the functions below
- `receive() external payable`
- `fallback() external payable`

`receive()` is called if `msg.data` is an empty value, and `fallback()` is used otherwise.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract ReceiveEther {
    /*
    Which function is called, fallback() or receive()?

           send Ether
               |
         msg.data is empty?
              / \
            yes  no
            /     \
receive() exists?  fallback()
         /   \
        yes   no
        /      \
    receive()   fallback()
    */

    // Function to receive Ether. msg.data must be empty
    receive() external payable {}

    // Fallback function is called when msg.data is not empty
    fallback() external payable {}

    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

contract SendEther {
    function sendEth(address payable _to) public payable {
        // Just forward the ETH received in this payable function
        // to the given address
        uint amountToSend = msg.value;
        // call returns a bool value specifying success or failure
        (bool success, bytes memory data) = _to.call{value: msg.value}("");
        require(success == true, "Failed to send ETH");
    }
}
```

## Calling External Contracts
Contracts can call other contracts by just calling functions on an instance of the other contract like `A.foo(x, y, z)`. To do so, you must have an interface for `A` which tells your contract which functions exist. Interfaces in Solidity behave like header files, and serve similar purposes to the ABI we have been using when calling contracts from the frontend. This allows a contract to know how to encode and decode function arguments and return values for calling external contracts.

Note: Interfaces you use do not need to be extensive. i.e. they do not need to necessarily contain all the functions that exist in the external contract - only those which you might be calling at some point. 

Assume there is an external `ERC20` contract, and we are interested in calling the `balanceOf` function to check the balance of a given address from our contract. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

interface MinimalERC20 {
    // Just include the functions we are interested in
    // in the interface
    function balanceOf(address account) external view returns (uint256);
}

contract MyContract {
    MinimalERC20 externalContract;
    
    constructor(address _externalContract) {
        // Initialize a MinimalERC20 contract instance
        externalContract = MinimalERC20(_externalContract);
    }
    
    function mustHaveSomeBalance() public {
        // Require that the caller of this transaction has a non-zero
        // balance of tokens in the external ERC20 contract
        uint balance = externalContract.balanceOf(msg.sender);
        require(balance > 0, "You don't own any tokens of external contract");
    }
}
```

## Import Statements
To maintain code readability, you can split your Solidity code over multiple files. Solidity allows importing both local and external files.

### Local Imports
Assume we have a folder structure like this:
```
├── Import.sol
└── Foo.sol
```

where `Foo.sol` is 
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Foo {
    string public name = "Foo";
}
```

We can import `Foo` and use it in `Import.sol` as such
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

// import Foo.sol from current directory
import "./Foo.sol";

contract Import {
    // Initialize Foo.sol
    Foo public foo = new Foo();

    // Test Foo.sol by getting it's name.
    function getFooName() public view returns (string memory) {
        return foo.name();
    }
}
```

NOTE: When we use Hardhat, we can also install contracts as node modules through `npm`, and then import contracts from the `node_modules` folder. These also count as local imports, as technically when you install a package you are downloading the contracts to your local machine.

### External Imports
You can also import from Github by simply copying the URL. We did this in the Cryptocurrency and NFT tutorials in the Freshman track.

```solidity
// https://github.com/owner/repo/blob/branch/path/to/Contract.sol
import "https://github.com/owner/repo/blob/branch/path/to/Contract.sol";

// Example import ERC20.sol from openzeppelin-contract repo
// https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol";

```

## Solidity Libraries
Libraries are similar to contracts in Solidity, with a few limitations. Libraries cannot contain any state variables, and cannot transfer ETH.

Typically, libraries are used to add helper functions to your contracts. An extremely commonly used library in Solidity world is `SafeMath` - which ensures that mathematical operations do not cause an integer underflow or overflow.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

library SafeMath {
    function add(uint x, uint y) internal pure returns (uint) {
        uint z = x + y;
        // If z overflowed, throw an error
        require(z >= x, "uint overflow");
        return z;
    }
}

contract TestSafeMath {
    function testAdd(uint x, uint y) public pure returns (uint) {
        return SafeMath.add(x, y);
    }
}
```

That's all folks! Congratulations on making it this far :D
