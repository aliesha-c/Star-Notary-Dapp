# Project 5: CryptoStar Dapp on Ethereum

This is the 5th project for the Blockchain Developer Nanodegree. The project is built upon the boiler plate code provided for a Star Notary service utilising the ERC-721 standard. It is deployed on the Rinkeby Ethereum Testnet with the following details:

Token Name: [**Star Deed**](https://rinkeby.etherscan.io/tokens?q=0x3f0e08b275De4975Bc1dF73F41180F2788155493)  
Token Symbol: **SDT**  
Contract Address: [**0x3f0e08b275De4975Bc1dF73F41180F2788155493**](https://rinkeby.etherscan.io/address/0x3f0e08b275de4975bc1df73f41180f2788155493)


## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

This project requires the following:
* [nodejs](https://nodejs.org) and npm (included in the nodejs install)
* [Truffle](https://truffleframework.com/truffle)
* [Metamask](https://metamask.io/)
* [Infura](https://infura.io)

To install truffle (after installing nodejs):
```
npm install truffle -g
```

### Installing

To install the project for review:
1. Download the project.
2. Run the command `npm install` to install the project dependencies.
3. Compile and deploy the contract in a local development environment:

```
> truffle develop
truffle(develop)> compile
truffle(develop)> migrate --reset
```

4. Deploy the Front End of the DAPP in a separate terminal window:

```
cd app
npm run dev
```

5. Open the front end by visiting [`http://127.0.0.1:8080`](http://127.0.0.1:8080)

## Running the tests

Run the tests for the contract by running the test command in the truffle development environment:

```
> truffle develop
truffle(develop)> test
```

## Deployment

To deploy the project on the rinkeby test network:
1. Save your mnemonic phrase in a file called `./.secret` 
2. Use truffle to deploy the contract:

```
truffle migrate --reset --network rinkeby
```

## Reviewing the Project

### Add contract token name and symbol
From `./contracts/StarNotary.sol`:
```
// Implement Task 1 Add a name and symbol properties
// Token name
string public constant name = "Star Deed";
// Token symbol
string public constant symbol = "SDT";
```

### Implement lookUptokenIdToStarInfo function
From `./contracts/StarNotary.sol`:
```
// Implement Task 1 lookUptokenIdToStarInfo
function lookUptokenIdToStarInfo(uint _tokenId) public view returns (string memory) {
    return tokenIdToStarInfo[_tokenId].name;
}
```

### Implement exchangeStars function
From `./contracts/StarNotary.sol`:
```
// Implement Task 1 Exchange Stars function
function exchangeStars(uint256 _tokenId1, uint256 _tokenId2) public {
    address ownerAddress1 = ownerOf(_tokenId1);
    address ownerAddress2 = ownerOf(_tokenId2);

    require(msg.sender == ownerAddress1 || msg.sender == ownerAddress2, 'Sender must own one of the stars');
    _transferFrom(ownerAddress1, ownerAddress2, _tokenId1);
    _transferFrom(ownerAddress2, ownerAddress1, _tokenId2);
}
```

### Implement transferStar function
From `./contracts/StarNotary.sol`:
```
// Implement Task 1 Transfer Stars
function transferStar(address _to1, uint256 _tokenId) public {
    address ownerAddress = ownerOf(_tokenId);

    require(msg.sender == ownerAddress, 'Sender must own star to transfer');
    _transferFrom(ownerAddress, _to1, _tokenId);
}
```

### Implement supporting unit tests
From `./test/TestStarNotary.js`:
```
// Implement Task 2 Add supporting unit tests
it('can add the star name and star symbol properly', async() => {
    let tokenId = 6;
    await instance.createStar('New Star!', tokenId, {from: accounts[0]});

    assert.equal(await instance.name.call(), "Star Deed");
    assert.equal(await instance.symbol.call(), "SDT");
});

it('lets 2 users exchange stars', async() => {
    let tokenId1 = 7;
    let tokenId2 = 8;
    await instance.createStar('New Star 1!', tokenId1, {from: accounts[0]});
    await instance.createStar('New Star 2!', tokenId2, {from: accounts[1]});

    instance.exchangeStars(tokenId1, tokenId2);

    assert.equal(await instance.ownerOf(tokenId1), accounts[1]);
    assert.equal(await instance.ownerOf(tokenId2), accounts[0]);
});

it('lets a user transfer a star', async() => {
    let tokenId = 9;
    await instance.createStar('New Star!', tokenId, {from: accounts[0]});

    instance.transferStar(accounts[1], tokenId);
    assert.equal(await instance.ownerOf(tokenId), accounts[1]);
});
```

### Deploy to Rinkeby
Deployment to Rinkeby requires the user's mnemonic phrase to be saved in a file called `./.secret`.

Configuration from `truffle-config.js`:
```
const infuraKey = "aea718a531d949c59204219998db790d";
const mnemonic = fs.readFileSync(".secret").toString().trim();
...
rinkeby: {
    provider: () => new HDWalletProvider(mnemonic, `https://rinkeby.infura.io/v3/${infuraKey}`),
    network_id: 4,       // rinkeby's id
    gas: 4500000,        // rinkeby has a lower block limit than mainnet
    gasPrice: 10000000000
}
```

### Implement lookUp function
From `./app/src/index.js`:
```
// Implement Task 4 Modify the front end of the DAPP
lookUp: async function () {
    const { lookUptokenIdToStarInfo } = this.meta.methods;
    const id = document.getElementById("lookid").value;
    const name = await lookUptokenIdToStarInfo(id).call();
    if (name == "") {
        App.setStatus(`Star ${id} has not been claimed`);
    } else {
        App.setStatus(`Star name is "${name}"`);
    }
}
```