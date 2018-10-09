# PM-JS usage
After you import pm-js as a dependency, you can initialize it by calling the method `create` returning a promise. But before doing that, let's install a web3 provider for our tests:
```sh
npm install truffle-hdwallet-provider-privkey ethereumjs-wallet
```
And generate a random private key for it:

```sh
export PRIVATE_KEY=$(node -e "console.log(require('ethereumjs-wallet').generate().getPrivateKey().toString('hex'))")
echo "Your private key: $PRIVATE_KEY"
export ADDRESS=$(node -e "console.log(require('ethereumjs-wallet').fromPrivateKey(Buffer.from('$PRIVATE_KEY', 'hex')).getChecksumAddressString())")
echo "Your address: $ADDRESS"
```

You can obtain rinkeby ETH using [their faucet](https://faucet.rinkeby.io/).

Once you have your rinkeby ETH, open your terminal and type: `node`
```javascript
const Gnosis = require('@gnosis.pm/pm-js')
const HDWalletProvider = require("truffle-hdwallet-provider-privkey");
let gnosis
if (!process.env){
    console.error("No PRIVATE_KEY env present")
    process.exit(1);
}

Gnosis.create(
    { ethereum: new HDWalletProvider([process.env.PRIVATE_KEY], "https://rinkeby.infura.io", 0, 1, false) }
).then(result => {
    gnosis = result
    // gnosis is available here and may be used
})

// note that gnosis is NOT guaranteed to be initialized outside the callback scope here
```
Create parameters:	
* `ethereum` (string|Provider) – An instance of a Web3 provider or a URL of a Web3 HTTP provider. If not specified, Web3 provider will be either the browser-injected Web3 (Mist/MetaMask) or an HTTP provider looking at http://localhost:8545
* `defaultAccount` (string) – The account to use as the default from address for ethereum transactions conducted through the Web3 instance. If unspecified, will be the first account found on Web3. See Gnosis.setWeb3Provider defaultAccount parameter for more info.
* `ipfs` (Object) – ipfs-mini configuration object
* * `ipfs.host` (string) – IPFS node address
* * `ipfs.port` (Number) – IPFS protocol port
* * `ipfs.protocol` (string) – IPFS protocol name
* logger (function) – A callback for logging. Can also provide ‘console’ to use console.log.


Know we would like to interact with a known market and perform buy/sell operations.

Let's instanciate the market:
```javascript
const market = gnosis.contracts.Market.at("0x5e9fe666935d274d3a85AD961CB1355BB576784B")
```

You can obtain also it's event contract:
```javascript
let event
market.eventContract().then(
    function (addr){
        event=gnosis.contracts.Event.at(addr) 
    }
)
```

Now we have the market and the event contract instances, we can perform all buy and sell mechanisms. Basically there are two ways of interacting with the prediction market outcome tokens:
1. Through the market contract and it's automated market maker (LMSR)
2. Buying all outcome tokens for later on use it with a custom market maker (your own automated market maker, an exchange, etc)

## Automated market maker

## Buy all outcomes
Buying all outcomes means exchange 1 collateral token (let's say WETH) to 1 Outcome token of each (Outcome Token YES, Outcome Token No for example). With this exchange of tokens you can always go back and exchange those again to collateral token if you use the function Sell All outcomes.

For all prediction markets we use ERC20 tokens, and because of this, all contract interaction needs to have an explicit approval of the tokens over the contract before you can actually buy/sell.

Let's try to buy all outcome tokens:
```javascript
async function buyAllOutcomes() {
    const depositValue = 1e17 // 0.1 ether
    const depositTx = await gnosis.etherToken.deposit.sendTransaction({ value: depositValue })
    await gnosis.etherToken.constructor.syncTransaction(depositTx)
    console.log("0.1 ETH deposited: https://rinkeby.etherscan.io/tx/" + depositTx)


    const approveTx = await gnosis.etherToken.approve.sendTransaction(event.address, depositValue)
    await gnosis.etherToken.constructor.syncTransaction(approveTx)
    console.log("0.1 WETH approved: https://rinkeby.etherscan.io/tx/" + approveTx)

    const buyTx = await event.buyAllOutcomes.sendTransaction(depositValue)
    await event.constructor.syncTransaction(buyTx)
    console.log("0.1 WETH exchanged 1:1 for collateral token index 0 and 1: https://rinkeby.etherscan.io/tx/" + depositTx)
}
buyAllOutcomes()
```

If you don't see errors in the terminal, the shares should have been bought. You can check your shares balance by executing this command:

