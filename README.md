# Time Locked Wallets

Blockchain and its applications are popular nowadays as never before.
Ethereum in particular, offering smart contract capabilities, opens the
doors to new ideas that can be implemented in a distributed, immutable,
and trustless fashion.

Getting started in the [Ethereum smart
contract](https://www.toptal.com/ethereum-smart-contract) space can be a
little overwhelming as the learning curve is quite steep. We hope that
this article (and future articles in the Ethereum series) can alleviate
this pain and get you up and running quickly.

Truffle, Solidity, and ĐApps {#truffle-solidity-and-apps}
----------------------------

In this article, we assume you have some basic understanding of
blockchain applications and Ethereum. If you feel like you need to brush
up your knowledge, we recommend [this Ethereum overview from the Truffle
framework](http://truffleframework.com/tutorials/ethereum-overview).

What’s covered in this post:

-   Applications of time-locked wallets
-   Development environment setup
-   Smart contract development with the Truffle framework
    -   Description of Solidity contracts
    -   How to compile, migrate, and test smart contracts
-   Using a ÐApp to interact with smart contracts from the browser
    -   Browser setup with MetaMask
    -   A run-through of the main use case

Time-locked Wallets: Use Cases
------------------------------

There are many different applications of
[Ethereum](https://www.toptal.com/ethereum) smart contracts. The most
popular at the moment are cryptocurrencies (implemented as ERC20 tokens)
and crowdfunding token sales (a.k.a. initial coin offerings, or ICOs.) A
good example of a utility ERC20 token is the [Motoro
Coin](https://www.toptal.com/ethereum/motoro-iot-in-transportation). In
this blog post, we will explore something different: The idea of locking
funds in crypto wallet contracts. This idea itself has various use
cases.

### Vesting for an ICO

There are several examples, but probably the most common reason at the
moment to lock funds is called “vesting.” Imagine that you have just
raised a successful ICO and your company still holds a majority of
tokens distributed between your team members.

It would be beneficial to all parties involved to ensure that the tokens
held by employees cannot be traded straightaway. If there are no
controls in place, any given employee might take action by selling all
their tokens, cashing out, and quitting the company. This would
negatively affect the market price and make all remaining contributors
to the project unhappy.

### Crypto-based “Last Will and Testament”

Another idea is to use a smart contract as a crypto-will. Imagine we
would like to store our cryptocurrency savings in a contract which will
be accessible by members of the family, but only after something has
happened to us. Let’s say we should “check in” with the wallet, by
evoking some contract call every so-often.

If we don’t check in on time, something presumably happened to us and
they can withdraw the funds. The proportion of funds they would each
receive could either be explicitly set in the contract, or it could be
left to be decided by consensus among the family members.

### A Pension or Trust Fund

Another application of locking funds could be to create a small pension
fund or time-based savings account, i.e., one that prevents the owner
from withdrawing any funds before a certain time in the future. (It
could be particularly useful for addicted crypto traders in helping keep
their ether intact.)

The use case we’ll explore for the rest of this blog post is similar: To
put some crypto-money away for later for someone else, like a future
birthday gift.

Let’s imagine we would like to gift one ether to someone for their 18th
birthday. We could write down on a piece of paper the account’s private
key and the address of the wallet holding the funds and hand it over to
them in an envelope. The only thing they would have to do is call a
function on the contract from their account once they are 18 and all the
funds will be transferred to them. Or, instead, we could just use a
simple ÐApp. Sounds good? Let’s get started!

Ethereum Development Setup
--------------------------

Before you forge ahead with smart contract development, you need to have
[Node.js](https://nodejs.org/en/) and [Git](https://git-scm.com/)
installed on your machine. In this blog, we are going to be using the
[Truffle](http://truffleframework.com/) framework. Even though you can
do without it, Truffle significantly reduces the entry barrier to the
development, testing, and deployment of Ethereum smart contracts. We
totally agree with their statement:

> *Truffle is the most popular development framework for Ethereum with a
> mission to make your life a whole lot easier.*

To install it, run the following command:

    npm install -g truffle

Now, get the code of this project:

    git clone https://github.com/radek1st/time-locked-wallets
    cd time-locked-wallets

It is important to note that the project follows the standard Truffle
project structure and the directories of interest are:

-   `contracts`: Holds all the Solidity contracts
-   `migrations`: Contains scripts describing the steps of migration
-   `src`: Includes the ÐApp code
-   `test`: Stores all the contract tests

Overview of the Included Smart Contracts
----------------------------------------

There are several contracts included in this project. Here’s the
rundown:

-   `TimeLockedWallet.sol` is the main contract of this project and it
    is described in detail below.
-   `TimeLockedWalletFactory.sol` is the `factory` contract which lets
    anyone easily deploy their own `TimeLockedWallet`.
-   `ERC20.sol` is an interface of the ERC20 standard for Ethereum
    tokens.
-   `ToptalToken.sol` is a customized ERC20 token.
-   `SafeMath.sol` is a small library used by ToptalToken for performing
    safe arithmetic operations.
-   `Migrations.sol` is an internal Truffle contract facilitating
    migrations.

For any questions on writing Ethereum contracts, please refer to the
official [Solidity smart contract
docs](http://solidity.readthedocs.io/en/develop/contracts.html).

### TimeLockedWallet.sol {#timelockedwalletsol}

Our `TimeLockedWallet.sol` Solidity contract looks like this:

    pragma solidity ^0.4.18;

The above line indicates the minimum version of the Solidity compiler
required for this contract.

    import "./ERC20.sol";

Here, we import other contract definitions, used later in the code.

    contract TimeLockedWallet {
        ...
    }

The above is our main object. `contract` scopes the code of our
contract. The code described below is from inside the curly brackets.

    address public creator;
    address public owner;
    uint public unlockDate;
    uint public createdAt;

Here we define several `public` variables which by default generate
corresponding getter methods. A couple of them are of type `uint`
(unsigned integers) and a couple are `address` (16-character long
Ethereum addresses.)

    modifier onlyOwner {
      require(msg.sender == owner);
      _;
    }

In simple terms, `modifier` is a precondition that has to be met before
even starting the execution of the function it is attached to.

    function TimeLockedWallet(
        address _creator, address _owner, uint _unlockDate
    ) public {
        creator = _creator;
        owner = _owner;
        unlockDate = _unlockDate;
        createdAt = now;
    }

This is our first function. As the name is exactly the same as our
contract name, it is the constructor and gets called only once when the
contract is created.

Note that if you were to change the name of the contract, this would
become a normal function callable by anyone and form a backdoor in your
contract like was the case in the [Parity Multisig Wallet
bug](https://medium.com/@codetractio/a-look-into-paritys-multisig-wallet-bug-affecting-100-million-in-ether-and-tokens-356f5ba6e90a).
Additionally, note that the case matters too, so if this function name
were in lowercase it would also become a regular function—again, not
something you want here.

    function() payable public { 
      Received(msg.sender, msg.value);
    }

The above function is of a special type and is called the `fallback`
function. If someone sends any ETH to this contract, we will happily
receive it. The contract’s ETH balance will increase and it will trigger
a `Received` event. To enable any other functions to accept incoming
ETH, you can mark them with the `payable` keyword.

    function info() public view returns(address, address, uint, uint, uint) {
        return (creator, owner, unlockDate, createdAt, this.balance);
    }

This is our first regular function. It has no function parameters and
defines the output tuple to be returned. Note that `this.balance`
returns the current ether balance of this contract.

    function withdraw() onlyOwner public {
       require(now >= unlockDate);
       msg.sender.transfer(this.balance);
       Withdrew(msg.sender, this.balance);
    }

The above function can only be executed if `onlyOwner` modifier defined
earlier is satisfied. If the `require` statement is not true, the
contract exits with an error. That’s where we check if the `unlockDate`
has gone by. `msg.sender` is the caller of this function and it gets
transferred the entire ether balance of the contract. In the last line,
we also fire a `Withdrew` event. Events are described a bit later.

Interestingly, `now`—which is equivalent to `block.timestamp`—may not be
as accurate as one may think. It is up to the miner to pick it, so it
could be up to 15 minutes (900 seconds) off as explained in the
following
[formula](https://github.com/ethereumproject/wiki/wiki/Block-Protocol-2.0#block-validation-algorithm):

> `parent.timestamp >= block.timestamp <= now + 900 seconds`

As a consequence, `now` shouldn’t be used for measuring small time
units.

    function withdrawTokens(address _tokenContract) onlyOwner public {
       require(now >= unlockDate);
       ERC20 token = ERC20(_tokenContract);
       uint tokenBalance = token.balanceOf(this);
       token.transfer(owner, tokenBalance);
       WithdrewTokens(_tokenContract, msg.sender, tokenBalance);
    }

Here is our function for withdrawing ERC20 tokens. As the contract
itself is unaware of any tokens assigned to this address, we must pass
in the address of the deployed ERC20 token we want to withdraw. We
instantiate it with `ERC20(_tokenContract)` and then find and transfer
the entire token balance to the recipient. We also fire a
`WithdrewTokens` event.

    event Received(address _from, uint _amount);
    event Withdrew(address _to, uint _amount);
    event WithdrewTokens(address _tokenContract, address _to, uint _amount);

In this snippet, we are defining several events. Triggered events are
basically log entries attached to the transaction receipts on the
blockchain. Each transaction can attach zero or more log entries. The
main uses of events are [debugging and
monitoring](https://docs.web3j.io/filters.html).

That’s all we need to time-lock ether and ERC20 tokens—just a few lines
of code. Not too bad, huh? Now let’s have a look at our other contract,
`TimeLockedWalletFactory.sol`.

### TimeLockedWalletFactory.sol {#timelockedwalletfactorysol}

There are two main reasons behind creating a higher-level factory
contract. The first one is a security concern. By separating the funds
in different wallets, we won’t end up with just one contract with a
massive amount of ether and tokens. This will give 100% of control to
just the wallet owner and hopefully discourage the hackers from trying
to exploit it.

Secondly, a factory contract allows easy and effortless TimeLockedWallet
contract creation, without the requirement of having any development
setup present. All you need to do is to call a function from another
wallet or ĐApp.

    pragma solidity ^0.4.18;

    import "./TimeLockedWallet.sol";

    contract TimeLockedWalletFactory {
        ...
    }

The above is straightforward and very similar to the previous contract.

    mapping(address => address[]) wallets;

Here, we define a `mapping` type, which is like a dictionary or a map,
but with all the possible keys preset and pointing to default values. In
the case of the `address` type, the default is a zero address `0x00`. We
also have an array type, `address[]`, which is holding `address`es.

In the Solidity language, Arrays always contain one type and can have a
fixed or variable length. In our case, the array is unbounded.

To sum up our business logic here, we define a mapping called `wallets`
which consists of user addresses—contract creators and owners alike—each
pointing to an array of associated wallet contract addresses.

    function getWallets(address _user) 
        public
        view
        returns(address[])
    {
        return wallets[_user];
    }

Here, we are using the above function to return all contract wallets a
`_user` created or has rights to. Note that `view` (in older complier
versions called `constant`) denotes that this is a function that doesn’t
change the blockchain state and can be therefore called for free,
without spending any gas.

    function newTimeLockedWallet(address _owner, uint _unlockDate)
        payable
        public
        returns(address wallet)
    {
        wallet = new TimeLockedWallet(msg.sender, _owner, _unlockDate);
        wallets[msg.sender].push(wallet);
        if(msg.sender != _owner){
            wallets[_owner].push(wallet);
        }
        wallet.transfer(msg.value);
        Created(wallet, msg.sender, _owner, now, _unlockDate, msg.value);
    }

This is the most important part of the contract: The factory method. It
lets us create a new time-locked wallet on the fly, by calling its
constructor: `new TimeLockedWallet(msg.sender, _owner, _unlockDate)`. We
then store its address for the creator and the recipient. Later we
transfer all the optional ether passed in this function execution to the
newly created wallet address. Finally, we signal the `Create` event,
defined as:

    event Created(address wallet, address from, address to, uint createdAt, uint unlockDate, uint amount);

### ToptalToken.sol {#toptaltokensol}

This tutorial wouldn’t be so much fun if we didn’t create our own
Ethereum token, so for completeness, we are bringing to life
`ToptalToken`. `ToptalToken` is a standard ERC20 token implementing the
interface presented below:

    contract ERC20 {
      uint256 public totalSupply;

      function balanceOf(address who) public view returns (uint256);
      function transfer(address to, uint256 value) public returns (bool);
      function allowance(address owner, address spender) public view returns (uint256);
      function transferFrom(address from, address to, uint256 value) public returns (bool);
      function approve(address spender, uint256 value) public returns (bool);

      event Approval(address indexed owner, address indexed spender, uint256 value);
      event Transfer(address indexed from, address indexed to, uint256 value);
    }

What distinguishes it from other tokens is defined below:

    string public constant name = "Toptal Token";
    string public constant symbol = "TTT";
    uint256 public constant decimals = 6;

    totalSupply = 1000000 * (10 ** decimals);

We gave it a name, a symbol, total supply of one million, and made it
divisible up to six decimals.

To discover different variations of token contracts, feel free to
explore the [OpenZeppelin
repo](https://github.com/OpenZeppelin/zeppelin-solidity/tree/master/contracts/token).

### Truffle Console: Compile, Migrate, and Test Smart Contracts

To get started quickly, run Truffle with the built-in blockchain:

    truffle develop

You should see something like this:

    Truffle Develop started at http://localhost:9545/

    Accounts:
    (0) 0x627306090abab3a6e1400e9345bc60c78a8bef57
    (1) 0xf17f52151ebef6c7334fad080c5704d77216b732
    (2) 0xc5fdf4076b8f3a5357c5e395ab970b5b54098fef
    (3) 0x821aea9a577a9b44299b9c15c88cf3087f3b5544
    (4) 0x0d1d4e623d10f9fba5db95830f7d3839406c6af2
    (5) 0x2932b7a2355d6fecc4b5c0b6bd44cc31df247a2e
    (6) 0x2191ef87e392377ec08e7c08eb105ef5448eced5
    (7) 0x0f4f2ac550a1b4e2280d04c21cea7ebd822934b5
    (8) 0x6330a553fc93768f612722bb8c2ec78ac90b3bbc
    (9) 0x5aeda56215b167893e80b4fe645ba6d5bab767de

    Mnemonic: candy maple cake sugar pudding cream honey rich smooth crumble sweet treat

The mnemonic seed lets you recreate your private and public keys. For
example, import it into MetaMask as shown here:

![RESTORE VAULT: Using a mnemonic wallet seed to import private and
public keys into
MetaMask.](https://uploads.toptal.io/blog/image/125509/toptal-blog-image-1519724583255-a3c7f959fc2b3ad9a7544e1e8aee87cd.png)

To compile the contracts, run:

    > compile

You should see:

    Compiling ./contracts/ERC20.sol...
    Compiling ./contracts/Migrations.sol...
    Compiling ./contracts/SafeMath.sol...
    Compiling ./contracts/TimeLockedWallet.sol...
    Compiling ./contracts/TimeLockedWalletFactory.sol...
    Compiling ./contracts/ToptalToken.sol...
    Writing artifacts to ./build/contracts

Now, we need to define which contracts we want to have deployed. This is
done in `migrations/2_deploy_contracts.js`:

    var TimeLockedWalletFactory = artifacts.require("TimeLockedWalletFactory");
    var ToptalToken = artifacts.require("ToptalToken");

    module.exports = function(deployer) {
      deployer.deploy(TimeLockedWalletFactory);
      deployer.deploy(ToptalToken);
    };

We first import our two contract artifacts `TimeLockedWalletFactory` and
`ToptalToken`. Then we simply deploy them. We missed `TimeLockedWallet`
on purpose, as this contract is deployed dynamically. For more
information on migrations, please refer to the [Truffle migrations
documentation](http://truffleframework.com/docs/getting_started/migrations).

To migrate the contracts, run:

    > migrate

This should result in something resembling the following:

    Running migration: 1_initial_migration.js
         Deploying Migrations...
         ... 0x1c55ae0eb870ac1baae86eeb15f3aba3f521df46d9816e04400e9b5951ecc099
         Migrations: 0x8cdaf0cd259887258bc13a92c0a6da92698644c0
       Saving successful migration to network...
         ... 0xd7bc86d31bee32fa3988f1c1eabce403a1b5d570340a3a9cdba53a472ee8c956
       Saving artifacts...
       Running migration: 2_deploy_contracts.js
         Deploying TimeLockedWalletFactory...
         ... 0xe9d9c37508bb58a1591d0f052d6870810118a0a19f728bf0cea4f4e5c17acd7a
         TimeLockedWalletFactory: 0x345ca3e014aaf5dca488057592ee47305d9b3e10
         Deploying ToptalToken...
         ... 0x0469ce110735f27bbb1a85c85a77ba4b0ba0d5aa52c3d67164045b849d8b2ed6
         ToptalToken: 0xf25186b5081ff5ce73482ad761db0eb0d25abfbf
       Saving successful migration to network...
         ... 0x059cf1bbc372b9348ce487de910358801bbbd1c89182853439bec0afaee6c7db
       Saving artifacts...

You can tell that `TimeLockedWalletFactory` and `ToptalToken` got
deployed successfully.

Finally, to ensure all is working well, let’s run some tests. The tests
are located in the `test` directory and correspond to the main contracts
`TimeLockedWalletTest.js` and `TimeLockedWalletFactoryTest.js`. For
brevity, we’ll not go into the details of writing tests, and leave it as
an exercise for the reader. To execute the tests, simply run:

    > test

…and hopefully you’ll see all the tests passing like this:

    Contract: TimeLockedWalletFactory
      ✓ Factory created contract is working well (365ms)

    Contract: TimeLockedWallet
      ✓ Owner can withdraw the funds after the unlock date (668ms)
      ✓ Nobody can withdraw the funds before the unlock date (765ms)
      ✓ Nobody other than the owner can withdraw funds after the unlock date (756ms)
      ✓ Owner can withdraw the ToptalToken after the unlock date (671ms)
      ✓ Allow getting info about the wallet (362ms)

    6 passing (4s)

Time-Locked Wallet ÐApp {#time-locked-wallet-app}
-----------------------

It’s time to see it all in action. The easiest way to interact with any
blockchain is by using distributed applications with a web UI, so called
ÐApps (or sometimes “dapps.”)

### Distributed Application Setup

In order to run this ÐApp, you will need to have an Ethereum-enabled
browser. The easiest way of achieving this is to install the
[MetaMask](https://metamask.io/) Chrome plugin. There’s also a [visual
guide on installing and configuring MetaMask with
Truffle](http://truffleframework.com/tutorials/pet-shop#interacting-with-the-dapp-in-a-browser).

### Smart Contract Scenario

Coming back to our scenario, why don’t we introduce the actors first?
Let’s assume Alice is going to be the creator of the time-locked wallet
and Bob will be the recipient/eventual owner of the funds.

![Accounts for Alice and Bob on
MetaMask.](https://uploads.toptal.io/blog/image/125505/toptal-blog-image-1519724560071-1d3329b07c87fc6f716f2591714de554.png)

Scenario Outline:

-   Alice creates a time-locked wallet for Bob and sends some ETH
-   Alice additionally sends some ERC20 Toptal Tokens
-   Bob can see the wallets he has access to and the ones he created
-   Bob cannot withdraw any funds before the wallet’s time lock expires
-   Bob withdraws the ETH when it gets unlocked
-   Bob withdraws all ERC20 Toptal Tokens

Firstly, Alice creates a time-locked wallet for Bob and sends an initial
one ether. We can see that a new contract wallet has been created and is
owned by Bob:

![Creating a wallet for Alice using the Time-Locked Wallets
ĐApp.](https://uploads.toptal.io/blog/image/125504/toptal-blog-image-1519724545675-5acadb74cd80d0c7dc0d064eca9d5c01.png)

At any point after the contract creation, the wallet can be topped up.
The top-up can come from anyone and be in form of ether or ERC20 tokens.
Let’s have Alice send 100 Toptal Tokens to Bob’s new wallet as depicted
here:

![Alice using the Time-Locked Wallets ĐApp to send Bob 100 Toptal
Tokens.](https://uploads.toptal.io/blog/image/125508/toptal-blog-image-1519724576723-a0d628883c52e945979722ba2d8b86ca.png)

From Alice’s point of view, the wallet will look like this after the
top-up:

![Alice's view of the topped-up
wallet.](https://uploads.toptal.io/blog/image/125507/toptal-blog-image-1519724570845-d8a9aac27a4aca1366639aece475f90a.png)

Let’s switch roles now and log in as Bob. Bob should be able to see all
the wallets he has created or is the recipient of. As the contract
created by Alice is still time-locked, he cannot withdraw any funds:

![Bob's view of the topped-up wallet while it is still
time-locked.](https://uploads.toptal.io/blog/image/125506/toptal-blog-image-1519724565763-a08c83154c19b27ac210cf63d0b31fa6.png)

Having patiently waited until the lock expired…

![A now-unlocked wallet on the Time-Locked Wallets
ĐApp.](https://uploads.toptal.io/blog/image/125510/toptal-blog-image-1519724588855-55e340b7ef7d1ecfb862fee1081dfae1.png)

…Bob is now ready to withdraw both ether and Toptal Tokens:

![Bob withdrawing ether from the unlocked
wallet.](https://uploads.toptal.io/blog/image/125511/toptal-blog-image-1519724595056-10aed68c19940b29041b4739bbfb6622.png)

![Bob withdrawing Toptal Tokens from the unlocked
wallet.](https://uploads.toptal.io/blog/image/125512/toptal-blog-image-1519724602976-c70c584e283545bf71c1f80f21f5c220.png)

After emptying the time-locked wallet his address balance has increased
and made him very happy and grateful to Alice:

![The unlocked wallet, now
empty.](https://uploads.toptal.io/blog/image/125503/toptal-blog-image-1519724540205-7ddc669430dc68e83e8a4098d13c05b9.png)

Please enable JavaScript to view the [comments powered by
Disqus.](https://disqus.com/?ref_noscript)
