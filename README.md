# Introduction

Towards the end of 2017, Gnosis hosted a prediction market tournament called [Olympia](https://blog.gnosis.pm/announcing-gnosis-olympia-5fb7e16dd259). It combined [GnosisDB](https://github.com/gnosis/gnosisdb), the [prediction market smart contracts](https://github.com/gnosis/pm-contracts), and the prediction market JS library in the context of a [user interface](https://github.com/gnosis/pm-trading-ui) we've been developing (we've also created a [separate repository](https://github.com/gnosis/olympia-interface) for the interface used in the original tournament run, but the work done there is deprecated, having been refactored into a feature flag for the generic interface).

Apollo will cover the configuration and operation of such a tournament. Here are the tools we will be starting out with:

* [npm](https://www.npmjs.com/)
* [Git](https://git-scm.com/)
* [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed for [GnosisDB](https://github.com/gnosis/gnosisdb).

Also, to run everything on a single machine, 8GB of RAM is required, and at least 16GB is recommended.

We will be assuming at least basic knowledge of the following:

* [Ethereum](https://www.ethereum.org/)
* [Solidity](https://github.com/ethereum/solidity)
* [Truffle](http://truffleframework.com/)

You can find the latest version of this manual at [ReadTheDocs](https://gnosis-apollo.readthedocs.io/en/latest/).
