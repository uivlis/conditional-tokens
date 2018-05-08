# Administration Via PM Scripts

Get the [Gnosis prediction market scripts](https://github.com/gnosis/pm-scripts). We will configure the utils in the following way:

`conf/config.json`: Let's configure the SDK utils use the `mnemonic` we used earlier to deploy the smart contracts. Also, make sure that the `collateralToken` is set to the deployed token. Finally, make sure that the `gnosisDB` instance is pointed at an instance configured for your tournament. For example:

```json
{
  "mnemonic": "romance spirit scissors guard buddy rough cabin paddle cricket cactus clock buddy",
  "account": "",
  "blockchain": {
    "protocol": "https",
    "host": "rinkeby.infura.io",
    "port": "443"
  },
  "gnosisDB": {
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

`conf/markets.json`: We list the markets on which we would like to operate here:

```json
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

Then, we use `node lib/main.js deploy` to deploy these markets to the network. These markets will then gain values in `conf/markets.json`:

```json
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

In order to allow tournament participants to take part in these markets, you will need to whitelist the `eventAddress` and `marketAddress` values. Go to your smart contract project (`big-token` in this example), and whitelist them with the following script:

```sh
npm run allow-transfers -- --network=rinkeby --addresses=0xf042bb28f521d02852dcc3635418a5cd7d9ab565,0xcb5f35384e268f37504beb2465c1b8f42be8f414,0xc04f5adc5deba8acb39c0fdf9db0f5ed8cfe270d,0x8bdc656a33ea8ee00e6fb7256bd9ea9e22ea7227
```

When a market has a winningOutcome, set its `winningOutcome` by editing `markets.json`:

```json
[
  {
    "title": "What will be the median gas price on December 31st, 2018?",
    ...,
    "winningOutcome": 0
  },
  ...
]
```

Then run `node lib/main.js resolve`, which will allow you to resolve those markets with winning outcomes set accordingly.
