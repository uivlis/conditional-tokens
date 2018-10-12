# Tournament Operator Guide
We assume, you have read past sections, and you already know how to operate a Gnosis prediction market platform: create markets, set up tradingdb, host the website and resolve the markets.
In this section we will explain step by steps what do you need in order to configure your own Prediction Markets Tournament.

# Set up contracts
## Create Ethereum Accounts
First of all, we need you to generate at least 2 accounts. Why two? because we need 1 account to issue tokens and other to create markets, and it might happen in parallel, so we need to separate the accounts to avoid nonce collision.
```sh
# In case you don't have ganache-cli. This is the main local testnet tool used for ethereum development. By default it creates random private keys and a Mnemonic, that's perfect for creating new accounts in bulk.

npm install -g 'ganache-cli'
ganache-cli
```

By executing this command, you get 10 accounts created, derived by a random mnemonic phrase and all it's related private keys, as you can see in the picture.
![Account Generation](img/accounts-ganache.png)

## Tournament Contracts
Download the 
```sh
git clone https://github.com/gnosis/pm-apollo-contracts.git
```

Usually you would like to rename the contract from 'OLY' to something related with your project. For that end you just need to change two lines:  

* [token name](https://github.com/gnosis/pm-apollo-contracts/blob/v1.4.1/contracts/OlympiaToken.sol#L9) 

* [token symbol](https://github.com/gnosis/pm-apollo-contracts/blob/v1.4.1/contracts/OlympiaToken.sol#L9)

You can also change the file itself, if you want for example to look with a different names when validating the contract on etherscan, but it's not necessary.

**In case you want to modify deeper the tournament token, the requirements are: It should be ERC20 compliant and also implement the issue function (if you want to use automatic issuance for new users)**

### Deploy
Set up your private key or mnemonic:
```sh
export MNEMONIC='client catch that man dice easily brave either fatal discover welcome tattoo'
# or export PRIVATEKEY='0xb7e68f153f86ebea910f834bb7488b1d843f782eb8eb12f3482813c69cd6c4aa'
```

Install dependencies and execute migration:
```sh
npm run migrate -- --network=rinkeby
# If you want to run it again, you need to add the option --reset
```

This command will deploy 3 contracts:
* `Truffle migration contract`. Keeps track of the different migrations, in case we add a new step, will go from the last point. Will not reset all the contracts.
* `Tournament Token`. It's the ERC20 token used by the tournament markets and users.
* `Address Registry`. It's the contract the users need to register to, in oder to appear in the scoreboard and also get tournament tokens (in case you set up auto-issuance).

### Validate Contracts
This step is completely optional, but it's a recommended practice, so you are transparent with your users about what the tournament contracts do.

Execute the command: 
```sh
npx truffle-flattener contracts/OlympiaToken.sol > ValidateToken.sol
npx truffle-flattener contracts/AddressRegistry.sol > ValidateRegistry.sol
```

So, now you should go to etherscan and validate both contracts, in the url `https://rinkeby.etherscan.io/verifyContract2?a=<address>`

being `<address>` the contract address, you can check those with:
```sh
npx truffle -- networks
```

You need to enter:
1. Contract Name: OlympiaToken or Address Registry
2. Compiler 0.4.23 commit (you can check it with `npx truffle version`)
3. Optimization off
4. Code, the content of `ValidateToken.sol` and `ValidateRegistry.sol` respectively.

### Configure Contracts.

Previously we created a bunch of ethereum accounts to separate nonce of the issuer and market creator and isolate roles. For that end we need to execute 2 transactions through the command line in the pm-contracts project.

```sh
export CREATOR_ADDRESS=<address>
npm run add-admins -- --addresses=$CREATOR_ADDRESS --network=rinkeby
npm run issue-tokens -- --amount 1000e18 --to $CREATOR_ADDRESS --network=rinkeby
```

**Note we issued 1000 Tournament tokens, it's in scientific notation. Represents 1000 units with 18 decimals (the default value for decimals)**

## Deploy Markets with pm-scripts
We assume you already take a look at [pm-scripts section](pm-scripts) and understand the usage of the tool. In order to deploy tournament markets you need to modify one more parameter in the `config.json`:
```javascript
"collateralToken": "<address>" # This is the Tournament Token Contract deployed before.
```

And also, as you are using a new account that has admin rights over the token, you need to set up that account in the `config.json`.

Before deploying the markets with `npm run deploy` you should see your Token Balance and validate the market information.

## TradingDB
You need to Set up the Indexer following the steps in [tradingdb section](pm-trading-db). As soon as you have set it up there are a few differences to configure it for tournaments. Basically now you have two more contract addresses and also an optional ethereum account (automatic token issuance).

You need to set up the following env params:
* `TOURNAMENT_TOKEN` Your tournament token contract.
* `ETHEREUM_DEFAULT_ACCOUNT_PRIVATE_KEY` Optional, ethereum private key of the token creator for automatic issuance.
* `GENERIC_IDENTITY_MANAGER_ADDRESS` Registry Contract.

There are other options available listed [here](pm-trading-db.html#tournament-token-issuance-olympia-related)

As soon as you configure your backend with these params, we need to create the periodic tasks and start the indexing, you can do it by executing the following command inside one of the containers (or in the root path if you are using a bare metal approach).
```sh
docker-compose run web sh
python manage.py setup_tournament --start-block-number
```

The command `setup_tournament` will prepare the database and set up periodic tasks:
  - `--start-block-number` will, if specified, start pm-trading-db processing at a specific block instead of all the way back at the genesis block. You should give it as late a block before tournament events start occurring as you can.
  - **Ethereum blockchain event listener** every 5 seconds (the main task of the application).
  - **Scoreboard calculation** every 10 minutes.
  - **Token issuance** every minute. Tokens will be issued in batches of 50 users (to prevent
  exceeding the block limitation). A flag will be set to prevent users from being issued again on next
  execution of the task.
  - **Token issuance flag clear**. Once a day the token issuance flag will be cleared so users will
  receive new tokens every day.

All these tasks can be changed in the [application admin](http://localhost:8000/admin/django_celery_beat/periodictask/).
You will need a superuser:

```sh
docker-compose run web sh
python manage.py createsuperuser
```

## Trading Interface
The prediction markets interface doesn't differ in terms of build process, but it does in the configuration. You need to enable the tournament functionality and specify who are the market creators, the tournament token, the registry contract and also how will the reward work (if present).
```sh
cd pm-trading-ui
NODE_ENV=production npm run build
```

### Configuration Template
First we need to generate the tournament template by running the command:
```
npm run build-config olympia/production
```
Adn then modify in `dist/config.js` the following parameters:
* `whitelist`: should have your market creator address
* `collateralToken`: Your Tournament Token address
* `scoreboard`: enabled
* `gameguide`: enabled

For the format of those parameters check the [interface section](pm-trading-ui.html#tournament-mode)

Now all the code over `dist/` it's ready to be served in your favourite web server.
## Market Resolution
Follows the same logic than regular markets. Check the resolution section [here](pm-scripts#resove-markets)

## Reward Claiming
If your tournament offers a reward for the TOP X in the scoreboard, you can send the reward manually, but maybe it's more practical to do it through the reward claiming contract we implemented, so you only need to perform two transactions, and you establish a time-frame for redeeming. After that timeframe you can claim it back those tokens that were not used.

This contract is part of `pm-apollo-contracts` repo. Anyone can deploy it, and it will be on mainnet, so be sure the account you pass as env parameter have enough ether to deploy the market (<0.1ETH).
```sh
cd pm-apollo-contracts
npx truffle exec scripts/deploy_reward_contract.js --token=<token-address> --network=mainnet
```
`token-address` is the token you use as reward for your tournament, can be any ERC20 token (e.g GNO, RDN, OMG...)

### Configure Reward Claiming on the Interface
Check [this example](pm-trading-ui.html#reward-claiming). You can define the dates from which the claiming will be available that won't be visible until you activate the claiming after the tournament ends.

### Enable Reward Claiming.
The account that created the contract is the only one that can enable the claiming. 
For setting it up, we use pm-scripts.
```sh
cd pm-scripts
```

In order to execute the Reward Claim feature the following configuration property must be added to the config.json file.
It specifies the Reward Claim contract address, the levels property, which defines the respective amount of winnings for each winner in the top X (number of levels in the array) positions from the scoreboard.
As the Reward Contract could be running on a different chain than the contracts, you have to specify the blockchain property as described below:

```
  "rewardClaimHandler": {
    "blockchain": {
      "protocol": "https",
      "host": "mainnet.infura.io",
      "port": "443"
    },
    "address": "0x42331cbc7D15C876a38C1D3503fBAD0964a8D72b",
    "duration": 86400,
    "decimals": 18,
    "levels": [
      { "value": 5, "minRank": 1, "maxRank": 1 },
      { "value": 4, "minRank": 2, "maxRank": 2 },
      { "value": 3, "minRank": 3, "maxRank": 3 },
      { "value": 2, "minRank": 4, "maxRank": 4 },
      { "value": 1, "minRank": 5, "maxRank": 5 },
      { "value": 0.9, "minRank": 6, "maxRank": 7 },
      { "value": 0.8, "minRank": 8, "maxRank": 9 },
      { "value": 0.7, "minRank": 10, "maxRank": 11 },
      { "value": 0.6, "minRank": 12, "maxRank": 13 },
      { "value": 0.5, "minRank": 14, "maxRank": 15 },
      { "value": 0.4, "minRank": 16, "maxRank": 17 },
      { "value": 0.3, "minRank": 18, "maxRank": 19 },
      { "value": 0.2, "minRank": 19, "maxRank": 34 },
      { "value": 0.1, "minRank": 34, "maxRank": 100 }
    ]
  }
 ```

 **Is important you define well duration (in seconds). This will be the timeframe your users have to redeem their tokens before you get can get back from the contract the remaining tokens.**
 
 To execute the Claim Reward just run the following command:
 
 ```sh
 npm run claimrewards
 ```