# Introduction

Towards the end of 2017, Gnosis hosted a prediction market tournament called [Olympia](https://blog.gnosis.pm/announcing-gnosis-olympia-5fb7e16dd259). It combined [pm-trading-db](https://github.com/gnosis/pm-trading-db), the [core smart contracts](https://github.com/gnosis/pm-contracts), and the pm-js library in the context of a [user interface](https://github.com/gnosis/pm-trading-ui) we've been developing. For making easier the deployment of prediction markets, we created a simple command line tool called [pm-scripts](https://github.com/gnosis/pm-scripts) that allows to create from simple prediction markets to complex futarchy markets.

We will be assuming at least basic knowledge of the following:

* [Ethereum](https://www.ethereum.org/)
* [Solidity](https://github.com/ethereum/solidity)
* [Truffle](http://truffleframework.com/)

## Documentation
Read the last documentation in: 
* [https://gnosis-apollo.readthedocs.io/en/latest/](https://gnosis-apollo.readthedocs.io/en/latest/).
It's a work in progress, so comments, suggestions, and collaborations are appreciated.

## Collaboration
Apollo it's composed by many open source projects with the goal of providing a foundational framework for building prediction market platforms.
Meet the community in the Gitter channel!
* [https://gitter.im/gnosis/Apollo](https://gitter.im/gnosis/Apollo)

## Generate the doc
```sh
virtualenv -p python3 env
. env/bin/activate
pip install -r requirements.txt
make livehtml
```