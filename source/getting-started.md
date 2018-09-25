# Getting Started

We are going to explain the minimum steps to have a prediction market deployed on a testnet and being able to interact with it through our trading interface.

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
You need to modify `conf/config.json` and use an ethereum account you own with ether. [check the rinkeby faucet](https://faucet.rinkeby.io/)

You can use this config:
```json
{  
   "accountCredential":"man math near range escape holiday monitor fat general legend garden resist",
   "credentialType":"mnemonic",
   "account":"0x7ec8664a7be9c96a7e8b627f84789e5850887312",
   "blockchain":{  
      "protocol":"https",
      "host":"rinkeby.infura.io/gnosis/",
      "port":"443"
   },
   "pm-trading-db":{  
      "protocol":"https",
      "host":"tradingdb.rinkeby.gnosis.pm",
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
**Please note that we are using a TEST ACCOUNT, don't use it with real funds or in production. Create another account with metamask, ganache-cli or any available ethereum wallet provider**

You can use the example market `pm-scripts/examples/categoricalMarket.json` or modify it's content. **Be careful with the date format** or it won't be indexed by the backend service.

Run the creation command:
```sh
npm run deploy -- -m examples/categoricalMarket.json -w 1e18
```

This will create all the contracts related with a prediction market, [wrap ether](https://weth.io/) for you and fund the market with the WETH.

Follow the instructions pm-script prompts on the console until the end.

# Run de Ethereum Indexer
There are many ways to run our ethereum indexer (trading-db) but let's start with the basic one.

Download the project:
```sh
git clone https://github.com/gnosis/pm-trading-db
cd pm-trading-db
docker-compose up
```
This will install all the dependencies and orchestrate the different docker containers declared in `docker-compose.yml`

It will take a few minutes to complete, depending on your network connection and computer resources.

Finally you will have the service running and a web server listening on http://localhost:8000/ , you can see here the documentation of the different endpoints that our trading interface uses.

By default the indexer points to the rinkeby network trough [Infura nodes](https://infura.io/)
