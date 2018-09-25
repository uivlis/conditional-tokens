## Smart Contracts Architecture
The Smart Contracts are designed in a modular manner in order to make it easy the integration with different Ethereum projects and extend its functionality. For example you could use Gnosis Smart Contracts for all trading functionality and use Augur as an Oracle by extending the Oracle interface and build an [Smart Contract Adapter](https://en.wikipedia.org/wiki/Adapter_pattern).

The main components are described in [this blog post](https://blog.gnosis.pm/getting-to-the-core-4db11a31c35f).

![Smart Contracts Architecture](https://cdn-images-1.medium.com/max/800/1*MIkHKEdWn9-KvhoT1Xk7Gg.png)

## Ethereum Indexer
Discovering data in ethereum it's complex and depending on the use case it might be even impossible.
The "standard way" to query bulk data in ethereum it's trough [filters](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethfilter) this is very convenient for discovering ERC20 tx's during a certaing period, or the creation of a Multisig contract trough certain factory, but imagine you want something more complex like: Get all markets created by certain ethereum address that use an specific token.
To get this information completely from ethereum, you will need to get all [MarketCreation events](https://github.com/gnosis/pm-contracts/blob/v1.1.0/contracts/Markets/StandardMarketFactory.sol#L27) and then query all the relations: Market -> Event -> Oracle -> IPFS

In practice this is `O(n^4)` and with many P2P network connections, might work with a few values, but as soon as you create markets, it will be unusable.

For this reason we have created an Ethereum Indexer called TradingDB a micro-service Python project that queries ethereum nodes and allows powerful queries to be run in milliseconds.

This is the basic architecture:
![TradingDB Architecture](img/tradingdb-diagram.jpg)

## Trading UI
Gnosis aims to provide an open framework uppon which build [different applications with many use cases](https://blog.gnosis.pm/the-power-of-prediction-markets-fedea0b71244) 

## Javascript Library

## Olympia Tournaments