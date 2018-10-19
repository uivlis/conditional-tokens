# Extending and Adding Wallet Providers

The Gnosis Apollo interface is built with extendability in mind, if you require a new integration for a wallet, that is not yet supported, here is how to get started.

This guide assumes you know how to set up the interface, and you have development tools set up.

Run the interface in development mode using `NODE_ENV=development npm run start`, this will make the interface available on `localhost:5000` where you can see your changes.

Our integrations reside in `src/integrations/`. There are a few existing ones you can take a look at, but here is how to create one from scratch.

### Extending `injectedWeb3`
If your provider simply injects web3, you can derive from the `InjectedWeb3` class in `src/integrations/injectedWeb3`.

Let's assume we want to integrate our imaginative wallet "SuperSecureWallet", it works like metamask but has a slightly different interface.

`src/integrations/supersecurewallet/index.js`
```
import { WALLET_PROVIDER } from 'integrations/constants'
import InjectedWeb3 from 'integrations/injectedWeb3'
import Web3 from 'web3'

class SuperSecureWallet extends InjectedWeb3 {
  static providerName = WALLET_PROVIDER.SUPER_SECURE_WALLET

  /**
   * Provider with highest priority starts off as active, if other providers are also available.
   * This allows "fallback providers" like a remote etherium host to be used as a last resort.
   */
  static providerPriority = 100

  /**
   * Tries to initialize and enable the current provider
   * @param {object} opts - Integration Options
   * @param {function} opts.runProviderUpdate - Function to run when this provider updates
   * @param {function} opts.runProviderRegister - Function to run when this provider registers
   */
  async initialize(opts) {
    super.initialize(opts)
    this.runProviderRegister(this, { priority: SuperSecureWallet.providerPriority })

    this.walletEnabled = false

    if (typeof window.web3 !== 'undefined' && window.web3.supersecure) {
      this.web3 = new Web3(window.web3.currentProvider)
      this.walletEnabled = true
    } else {
      this.walletEnabled = false
    }

    if (this.walletEnabled) {
      // The following functions all work with the default metamask/web3 interface
      this.networkId = await this.getNetwork()
      this.network = await this.getNetwork()
      this.account = await this.getAccount()
      this.balance = await this.getBalance()
    }

    return this.runProviderUpdate(this, {
      available: this.walletEnabled && this.account != null,
      networkId: this.networkId,
      network: this.network,
      account: this.account,
      balance: this.balance,
    })
  }
}
export default new SuperSecureWallet()
```

A few things are missing now, add your integration to the integration-exporter in `/src/integrations/index.js`
```js
...
import { WALLET_PROVIDER } from './constants'
import SuperSecureWallet from './supersecurewallet'
...

const providers = {
  [WALLET_PROVIDER.SUPERSECUREWALLET]: SuperSecureWallet,
  [WALLET_PROVIDER.METAMAS]: Metamask,
  [WALLET_PROVIDER.PARITY]: Parity,
  [WALLET_PROVIDER.REMOTE]: Remote
}
```

The provider constants are defined in `/src/integrations/constants.js`:
```js
export const WALLET_PROVIDER = {
  SUPERSECUREWALLET: 'SUPERSECUREWALLET',
  METAMASK: 'METAMASK',
  PARITY: 'PARITY',
  REMOTE: 'REMOTE',
  UPORT: 'UPORT',
}
```
(These are used internally to define parts in our `redux` storage)

Add your own provider icon for the header component as `src/assets/img/icons/icon_<provider icon variable>.svg`, so in our case you'll need to define `icon_supersecurewallet.svg`. Take a look at our icons for inspiration and to stay within a consistent style.

This is everything needed to set up a new web3-based provider, making your provider available will show it in the header section, if you are connected to the interface.

Please note that if you want to supply a non-web3 provider, it still needs to have the interface to interact with contracts instanciated with `truffle-contract`, e.g. this example still has to work:
```js
const TruffleContract = require('truffle-contract')
const MyContractArtifact = require('./build/contracts/MyContract.json')
const MyContract = TruffleContract(MyContractArtifact)
MyContract.setProvider(myProvider)
```
For more information on the `truffle-contract` interface, read up on [their repository](https://github.com/trufflesuite/truffle-contract).
