# PM-SCRIPTS
pm-scripts is the recommended tool for deploying your prediction markets contracts. It allows you to deploy all kinds of prediction markets easily in any network even without having a full understanding of what all the pieces of a prediction market are. 

Let's start by getting the [pm-scripts](https://github.com/gnosis/pm-scripts). 
```sh
git clone https://github.com/gnosis/pm-scripts
npm i
```

We will configure the utils in the following way:

## Configuration

`conf/config.json`: Let's configure the pm-scripts using the `mnemonic` we used earlier to deploy the smart contracts. Also, make sure that the `collateralToken` is set to the deployed token. Finally, make sure that the `tradingDB` instance is pointed at an instance configured for your tournament. For example:

```js
{
  "mnemonic": "romance spirit scissors guard buddy rough cabin paddle cricket cactus clock buddy",
  "account": "",
  "blockchain": {
    "protocol": "https",
    "host": "rinkeby.infura.io",
    "port": "443"
  },
  "tradingDB": {
    "protocol": "http",
    "host": "localhost",
    "port": "8001"
  },
  "ipfs": {
    "protocol": "http",
    "host": "localhost",
    "port": "5001"
  },
  "gasPrice": "1000000000",
  "collateralToken": "0x0152b7ed5a169e0292525fb2bf67ef1274010c74"
}
```

* **accountCredential**: This is your wallet credential. Can be either an HD wallet mnemonic phrase composed by 12 words ([HD wallet repository](https://github.com/trufflesuite/truffle-hdwallet-provider)) or a private key ([HD wallet private key repository](https://github.com/rhlsthrm/truffle-hdwallet-provider-privkey));
* **credentialType**: is the type of credential you want to use to access your account, available values: `mnemonic`, `privateKey`, default is `privateKey`;
* **account**: is your ethereum address, all transactions will be sent from this address. If not provided, pm-scripts will calculate it from your mnemonic phrase;
* **blockchain**: defines the Ethereum Node that pm-scripts should send transactions to (https://rinkeby.infura.io/gnosis/ by default);
* **tradingDB**: defines the [pm-trading-db](pm-trading-db/) url, an Ethereum indexer which exposes a handy API to get your list of markets and their details (default: https://tradingdb.rinkeby.gnosis.pm:443);
* **ipfs**: sets the IPFS node that pm-scripts should send transactions to (https://ipfs.infura.io:5001 by default);
* **gasPrice**: the desired gasPrice
* **collateralToken**: the Collateral Token contract's address (e.g Ether Token):
  - **Rinkeby:** [0xc778417e063141139fce010982780140aa0cd5ab](https://rinkeby.etherscan.io/address/0xc778417e063141139fce010982780140aa0cd5ab)
  - **Kovan:** [0xd0a1e359811322d97991e03f863a0c30c2cf029c](https://kovan.etherscan.io/address/0xd0a1e359811322d97991e03f863a0c30c2cf029c)
  - **Ropsten:** [0xc778417e063141139fce010982780140aa0cd5ab](https://ropsten.etherscan.io/address/0xc778417e063141139fce010982780140aa0cd5ab)
  - **Mainnet:** [0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2](https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2)

## Deploy markets

`conf/markets.json`: We list the markets on which we would like to operate here:

```js
[
  {
    "title": "What will be the median gas price on December 31st, 2018?",
    "description": "What will be the median gas price payed among all transactions on December 31st, 2018?",
    "resolutionDate": "2018-12-31T18:00:00.000Z",
    "outcomeType": "CATEGORICAL",
    "outcomes": [
      "< 20 GWEI",
      "20 GWEI",
      "> 20 GWEI"
    ],
    "currency": "WETH",
    "fee": "0",
    "funding": "1e18"
  },
  {
    "title": "What will the expected volatility of the Ethereum market be by December 31st over a 30-day estimate?",
    "description": "What will the expected volatility of the Ethereum market be by December 31st, 2018, over a 30-day estimate? Source: https://www.buybitcoinworldwide.com/ethereum-volatility/",
    "resolutionDate": "2018-12-31T18:00:00.000Z",
    "outcomeType": "SCALAR",
    "upperBound": "11",
    "lowerBound": "2",
    "decimals": 0,
    "unit": "%",
    "currency": "WETH",
    "fee": "0",
    "funding": "1e18"
  }
]
```
### Params
* **title**: The title of the market.

* **description**: A text field describing the title of the market.

* **resolutionDate**: Defines when the prediction market ends, you can always resolve a market before its resolutionDate expires.
Format must be any recognised by
[Javascript Date constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date),
it's recommended to use an ISO date format like *2018-03-27T16:20:11.698Z*.

* **currency**: A text field defining which currency is holding the market's funds. It's informative, just to remind you wich token corresponds to the collateral token address.

* **fee**: A text field defining the amount of fees charged by the market creator.

* **funding**: A text field representing how much funds to provide the market with. (e.g 1e18 == 1 WETH, 1e19 == 10 WETH...)

* **winningOutcome**: A text field representing the winning outcome. If declared, pm-scripts  will try to resolve the market, but will always ask you to confirm before proceeding.

* **outcomeType**: Defines the prediction market type. You must strictly provide 'CATEGORICAL' or 'SCALAR' (categorical market
or scalar market).

* **upperBound**: (scalar markets) A text field representing the upper bound of the predictions range.

* **lowerBound**: (scalar markets) A text field representing the lower bound of the predictions range.

* **decimals**: (scalar markets) Values are passed in as whole integers and adjusted to the right order of magnitude according to the decimals property of the event description, which is a numeric integer.

* **unit**: (scalar markets) A text field representing the market's unit of measure, like '%' or '°C' etc...

* **outcomes**: (categorical markets) An array of text fields representing the available outcomes for the market.


Then, we use `npm run deploy` to deploy these markets to the network. These markets will then gain values in `conf/markets.json`:

```js
[
  {
    "title": "What will be the median gas price on December 31st, 2018?",

    ...,

    "oracleAddress": "0xebf5e8897c15f3350fc3dc3032484dff7916dc75",
    "ipfsHash": "QmXqkAe1oBP2z2xLe7h8hCcKSrbb5LFDRr9zHkApdz3Xyh",
    "eventAddress": "0xf042bb28f521d02852dcc3635418a5cd7d9ab565",
    "marketAddress": "0xcb5f35384e268f37504beb2465c1b8f42be8f414"
  },
  {
    "title": "What will the expected volatility of the Ethereum market be by December 31st over a 30-day estimate?",

    ...,

    "oracleAddress": "0x1c959692196025cd3e95c1c4661c94366de23612",
    "ipfsHash": "QmY3LDEp2Hz7c9iM8Jci83VurhTkZWW7ccGr8WfNtRf8Rj",
    "eventAddress": "0xc04f5adc5deba8acb39c0fdf9db0f5ed8cfe270d",
    "marketAddress": "0x8bdc656a33ea8ee00e6fb7256bd9ea9e22ea7227"
  }
]
```

## Resolve Markets
pm-scripts is also used to resolve the outcome of a market. For that the steps are easy, let's set up the parameter `winningOutcome`:
```javascript
[
  {
    ...
    "winningOutcome": 123456789
    ...
  }
]
```
And press `resolve`:
```sh
npm run resolve
```
