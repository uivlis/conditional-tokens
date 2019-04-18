# Getting Started

In what follows, we will explain the basic steps to deploy a prediction market on the testnet, and how to interact with it through our trading interface.

## Requirements
* [Nodejs >= 7](https://nodejs.org/en/)
* [NPM >= 5](https://nodejs.org/en/)
* [Git](https://git-scm.com/downloads)
* [Docker Compose >= 3.6](https://docs.docker.com/compose/install/)
* [Docker >= 18.02](https://docs.docker.com/install/)

## Create the market
```
git clone https://github.com/gnosis/pm-scripts
cd pm-scripts
npm i
````
You need to modify `conf/config.json` and use an ethereum account you own which has ether. [check the rinkeby faucet](https://faucet.rinkeby.io/)

You can use the example configuration below that comes with the project for our test account:

**Don't use the test account with real funds or in production. Create another account for yourself with metamask, ganache-cli or another ethereum wallet provider, and make sure you change the credentials to match that account.**


```json
{  
   "accountCredential":"man math near range escape holiday monitor fat general legend garden resist",
   "credentialType":"mnemonic",
   "account":"0x7ec8664a7be9c96a7e8b627f84789e5850887312",
   "blockchain":{  
      "protocol":"https",
      "host":"rinkeby.infura.io",
      "port":"443"
   },
   "pm-trading-db":{  
      "protocol":"https",
      "host":"tradingdb.rinkeby.gnosis.io",
      "port":"443"
   },
   "ipfs":{  
      "protocol":"https",
      "host":"ipfs.infura.io",
      "port":"5001"
   },
   "gasPrice":"1000000000",
   "collateralToken":"0xd19bce9f7693598a9fa1f94c548b20887a33f141"
}
```

You can use the example market `pm-scripts/examples/categoricalMarket.json` or modify its content. **Be careful with the date format**. Otherwise it won't be indexed by the backend service.

Run the creation command:
```sh
npm run deploy -- -m examples/categoricalMarket.json -w 1e18
```

This will create all the contracts related to a prediction market, [wrap some ether](https://weth.io/) and fund the market with the newly created WETH.

Follow the instructions that `pm-script` prompts in the console until the end.

## Run the Ethereum Indexer
There are many ways to run our ethereum indexer (trading-db) but let's start with the most basic one.

Download the project:
```sh
git clone https://github.com/gnosis/pm-trading-db
cd pm-trading-db
docker-compose up
```
This will install all the dependencies and orchestrate the different docker containers declared in `docker-compose.yml`

It will take a few minutes to complete, depending on your network connection and computational resources.

Finally you will have the service running and a web server listening on [http://localhost:8000/](http://localhost:8000/) , you can see here the documentation of the different endpoints that our trading interface uses.

By default the indexer points to the rinkeby network through [Infura nodes](https://infura.io/). Indexing a full chain can take a few hours consuming all nodes resources, but we don't need to index the entire blockchain. We just need start the indexing process since the block which includes our prediction market contracts.

If you created the market now, you can substract a few blocks from the current block. Go to [etherscan](https://rinkeby.etherscan.io/) substract 100 blocks (that's around 20min of blocks) and execute:
```
docker-compose run web python manage.py setup --start-block-number <your-block-number>
```

This will start the indexing of the rinkeby chain and should take a few seconds. You should now see your market indexed in [http://localhost:8000/api/markets/](http://localhost:8000/api/markets/)

**Note: the default configuration points to infura and is very light in terms of performance so the service is not rate limited. For production settings, use `DJANGO_SETTINGS_MODULE=config.settings.production`**

## Setup the interface
The trading-ui interface offers a generic interface to interact with prediction markets. It is intended to be used as the starting point which can be extended for your use case.
Let's start downloading and installing the project:
```
git clone https://github.com/gnosis/pm-trading-ui
cd pm-trading-ui
docker-compose build --force-rm
```

The interface is already functional, but we need to configure it with our ethereum account as a whitelisted account, in order to display the markets in the interface.
Let's build the config template:
```
docker-compose run web npm run build-config
```

open the file `dist/config.json` with your favourite text editor and change `whiteslist: {}` for something like this:
```json
whitelist: {
    "operator": "<your-ethereum-address>"
}
```

Now everything is set, you can run the interface and start buying shares on your first prediction market! run `docker-compose up` and open your browser at [http://localhost:5000](http://localhost:5000)
