# Prediction markets as modular framework
The prediction markets platform that Gnosis offers, aims to provide the foundational protocol upon which many projects will grow using prediction markets as one small piece or a as a core part of their projects.
In this section we describe the different layers that compose the prediction markets framework.

## Trading UI
The generic interface to interact with prediction markets is trading-ui, a javascript project built with React that you can use as starting point to adapt it for [your particular use case](https://blog.gnosis.pm/the-power-of-prediction-markets-fedea0b71244) 


## Ethereum Indexer
Discovering data in ethereum it's complex and depending on the use case it might be even impossible.
The "standard way" to query bulk data in ethereum it's trough [filters](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethfilter) this is very convenient for discovering ERC20 tx's during a certaing period, or the creation of a Multisig contract trough certain factory, but imagine you want something more complex like: Get all markets created by certain ethereum address that use an specific token.
To get this information completely from ethereum, you will need to get all [MarketCreation events](https://github.com/gnosis/pm-contracts/blob/v1.1.0/contracts/Markets/StandardMarketFactory.sol#L27) and then query all the relations: Market -> Event -> Oracle -> IPFS

In practice this is `O(n^4)` and with many P2P network connections, might work with a few values, but as soon as you create markets, it will be unusable.

For this reason we have created an Ethereum Indexer called TradingDB a micro-service Python project that queries ethereum nodes and allows powerful queries to be run in milliseconds.

This is the basic architecture:
![TradingDB Architecture](img/tradingdb-diagram.jpg)

## Javascript Library
If you want to go deeper and integrate with the ethereum blockchain, our javascript library `pm-js` it's the middleware between the Smart Contracts and your program. It abstracts you away some of the logic related with prediction markets and adds some useful features like validation.

You can use directly the contracts with web3, and it could be more intuitive for you but will be harder to perform buy and sell operations and also web3 won't validate your parameters so you will have to be careful with the values you use or there will be many failing transactions.


## Smart Contracts
The Smart Contracts are designed in a modular manner in order to make it easy to integrate with different Ethereum projects and extend its functionality. For example you could use Gnosis Smart Contracts for all trading functionality and use Augur as an Oracle by extending the Oracle interface and build an [Smart Contract Adapter](https://en.wikipedia.org/wiki/Adapter_pattern).

The main components are described in [this blog post](https://blog.gnosis.pm/getting-to-the-core-4db11a31c35f).

![Smart Contracts Architecture](https://cdn-images-1.medium.com/max/800/1*MIkHKEdWn9-KvhoT1Xk7Gg.png)