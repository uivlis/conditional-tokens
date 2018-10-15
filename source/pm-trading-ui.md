# Setting Up the Interface

First, [clone the interface from github](https://github.com/gnosis/pm-trading-ui):
`git clone https://github.com/gnosis/pm-trading-ui.git`

## Setup

Run `npm i` to install all dependencies. Now you can already start the application like such: `NODE_ENV=development npm start` and it will run a local webpack server that you can test on.

In order to build a production version, run `NODE_ENV=production npm build` and it will create a `/dist` folder that will be filled with a minified and bundled interface application.

But first let's configure the interface:

## Configuration

The *pm-trading-ui* uses a **runtime configuration** so that we have the ability to deploy changes to configuration at runtime. It works by having a default *fallback* configuration, and multiple different environments and configuration files.

Let's look at a practical example, let's say we want to setup an automated "staging" or pre-production environment. For this example I'll use our mainnet configuration example:

```
/config
├── fallback.json  # the default configuration file, don't edit this file!
├── local.json
├── mainnet
│   ├── development.json
│   ├── production.json
│   └── staging.json # our desired environment configuration, do edit this file
└── olympia
    ├── production.json
    └── staging.json
```

You only need to define what you need in the specific configuration. Everything that is not defined, will be taken from `fallback.json`, which will run only the bare minimum of features.

Now, let's build and deploy our application. First, we need to create the `/dist` folder by running `NODE_ENV=production npm build`. Now we need to copy our desired configuration into our build folder in a special way as `config.js`. This method allows us to easily exchange it later on, either automated using a CI system or manual, without having to build everything again.

In order to copy the configuration file, we prepared a script that will copy and prepare the configuration file automatically. 
`node ./scripts/configuration.browser.js mainnet/staging`

`configuration.browser.js` is a simply script to copy, minify and add a crucial `window.__GNOSIS_CONFIG__=` snippet infront of your JSON config (as you can't embed JSON as script files in HTML)

After this step, your application is ready to be run. Deploy your application to your filehoster of choice and access the page - the previously mentioned `config.js` will be used to determine the configuration, at runtime.

## Basic Configuration Documentation

A quick rundown of all configuration entries, their meanings and their possible values. Please note that the interface currently does not throw warnings or errors, if you mistype a config entry, and will probably just use the fallback configuration.

### Trading DB
`gnosisdb` configures which trading-db you want to use to run the interface. The `pm-trading-db` package is required in order to keep track of previous markets, without having to fully sync an ethereum node, each time you want to access the interface.


`protocol` - either `https` or `http`

`host` - hostname for the database.

`port` - 443 is the default for SSL.


```js
{
  "gnosisdb": {
    "protocol": "https",
    "host": "example.com/trading-db",
    "port": 443
  },
```

### Ethereum Node
`ethereum` configures which ethereum node should be used to interact with the application. Infura is what we use and what is tested most in depth, but all other full-nodes should work too.

`protocol` - either `https` or `http`

`host` - hostname for the database.

`port` - 443 is the default for SSL.


```js
  "ethereum": {
    "protocol": "https",
    "host": "rinkeby.infura.io",
    "port": 443
  },
```

### Gas Price Calculation
In order to display the cost of transactions, we require an external gas-estimation service. Multiple different ones are availble, [ETHGasstation](https://ethgasstation.info) is the default but you can also define your own ([take a look at the code](https://github.com/gnosis/pm-trading-ui/blob/master/src/api/gasPrice.js#L16)).


`external.url` - the API url from which to fetch the gas price information

`external.type` - Which implementation does the API use? currently only available `ETH_GAS_STATION` but extendable, as mentioned above.

```js
  "gasPrice": {
    "external": {
      "url": "https://ethgasstation.info/json/ethgasAPI.json",
      "type": "ETH_GAS_STATION"
    }
  },
```
If you rather use the built-in gas estimation, which is supectible to gas-price attacks, define this entry as such:
```js
  "gasPrice": {
    "external": false
  }
```

### Market Creator Whitelist
The whitelist defines which users are allowed to create markets on your interface. Currently there is no way to disable the whitelist.

The object keys define the allowed addresses, the values (currently unused) are simply used as a way to remember which address belongs to which user. **Please enter all addresses (the keys) in lowercase**
```js
  "whitelist": {
    "0x123...": "Admin #1"
  },
```

### Logo and Favicon
This property defines which icons the interface should use for differenct screensizes and as a favicon. All paths are defines from the root of the /src folder

```js
  "logo": {
    "regular": "assets/img/gnosis_logo.svg",
    "small": "assets/img/gnosis_logo_icon.svg",
    "favicon": "assets/img/gnosis_logo_favicon.png"
  },
```

### Tournament Mode
Enabling Tournament Mode will currently enable the following functionality
- Custom Application Name will be used
- Gamification Stats on `/dashboard` Page
- Scoreboard, if desired
- Gamerules, if desired

```js
  "tournament": {
    "enabled": false,
    "name": "My Tournament"
  },
```

#### Scoreboard
If you want to use a scoreboard in your application, please take a look at [`pm-trading-db`](https://github.com/gnosis/pm-trading-db#tournament-setup).
```js
  "scoreboard": {
    "enabled": false
  },
```

#### Gameguide
The gameguide allows you to set rules and information for new users. In Olympia this is used to tell the user, how to use the interface if they're new to ethereum and the blockchain.
```js
  "gameGuide": {
    "enabled": false
  },
```

### Define a Collateral Token
You can define which collateral token the application should use when interacting with markets. **Setting this property will also filter all markets based on their collateral, meaning only markets with the same collateral token as the one that was defined here will be shown!**

`source` - defines how you want the ERC20 token contract should be found

  `contract` - means you define a contract thats available in `pm-js`s `Contracts` property. [To implement this, take a look at how this was done for our *olympia* tournament contracts.](https://github.com/gnosis/pm-trading-ui/blob/master/src/api/gnosis.js#L15)

  `address` - hardcoded address of the contract that's available on the network defined in `ethereum`. This is probably the easiest to setup.

  `eth` - uses a combination of Ether and `WETH`, a ERC20 wrapped Ether token. [Take a look here, for more information on this contract](https://github.com/gnosis/pm-contracts/blob/master/contracts/Tokens/EtherToken.sol)

`contractName` - is only required when using `source: "contract"`, defines the name of the contract to be loaded.

`symbol` - is used to overwrite the symbol. If this is not defined, it will try to use the name of the ERC20 token after loading it.

`icon` - used to display next to the amount of collateral a user has. If not defined, will use a default `ethereum` style icon.


```js
  "collateralToken":  {
    "source": "contract",
    "options": {
      "contractName": "etherToken",
      "symbol": "ETH",
      "icon": "/assets/img/icons/icon_etherTokens.svg"
    }
  },
```

### Wallet Integrations

There are multiple different built-in providers that can be used with the interface. The most tested provider is metamask. [Take a look at the code in order to build your own.](https://github.com/gnosis/pm-trading-ui/tree/master/src/integrations). Currently the following providers are available: `parity`, `metamask`, `remote`, `uport`. All providers are always available, as long as the correct network is used.

`default` - defines which provider to use when multiple providers were found, or if no provider was found to tell the user which provider is recommended to interact with the application.

`requireTOSAccept` - if you require the user to accept the terms and conditions before they can connect to the application and interact with it.

```js
  "providers": {
    "default": "METAMASK",
    "requireTOSAccept": false
  },
```

### Legal Compliance

If you require legal compliance when your application is used outside of an internal testing or similar situations, you can enable a feature which will attach itself in multiple places, to have the user check-off all defines documents before they can access the application.

`documents[].type` - defines the type of legal information you want the user to accept. Can be either 

`TOS_DOCUMENT` to refer to a `file` or `TOS_TEXT` to allow you to simply enter a text as `text`

`documents[].id` - defines a unique identifier that is used to check if the user previously accepted this document. If you later update a legal document, a version can be added, that will require the user to accept a specific document again. e.g. `terms_of_service_v2`.

`documents[].title` - Only for `TOS_DOCUMENT` to display as the title of the linked document.

`documents[].file` - Only for `TOS_DOCUMENT` to link to a document.

`documents[].text` - Only for `TOS_TEXT` to insert text that the user will have to "agree to"


```js
  "legalCompliance": {
    "enabled": true,
    "documents": [
      {
        "type": "TOS_DOCUMENT",
        "id": "terms_of_service",
        "title": "Terms of Service",
        "file": "/assets/TermsOfService.html"
      },
      {
        "type": "TOS_DOCUMENT",
        "id": "privacy_policy",
        "title": "Privacy Policy",
        "file": "/assets/PrivacyPolicy.html"
      },
      {
        "type": "TOS_DOCUMENT",
        "id": "risk_disclaimer",
        "title": "Risk Disclaimer",
        "file": "/assets/RiskDisclaimerPolicy.html"
      },
      {
        "type": "TOS_TEXT",
        "id": "cookie_policy",
        "text": "Gnosis' Cookies",
      }
    ]
  },
```


### Page Footer
You can define a custom footer, using either a `file` or `text`, as such:

`footer.content.type` - either `text` or `file`

`footer.content.fileName` - Only for `file`, the name of the file in `/assets/content`, has to end with `.md`

`footer.content.source` - Only for `text`, the content of the footer as markdown.

`footer.content.markdown` - Enabled markdown parsing of either the file or `source`, is type `text` is defined.


```js
  "footer": {
    "enabled": true,
    "content": {
      "type": "file",
      "fileName": "footer",
      "markdown": true
    }
  },
```

### Reward Claiming
[See here](./pm-trading-ui-rewards.html)
```js
  "rewardClaiming": {
    "enabled": false,
    "claimReward": {
      "enabled": false,
      "claimStart": "2018-06-01T12:00:00",
      "claimUntil": "2018-07-01T12:00:00",
      "contractAddress": "0xe89f27dafb9ba68c864e47a0bf1e430664e419af",
      "networkId": 42
    }
  },
  "rewards": {
    "enabled": false,
    "rewardToken": {
      "symbol": "RWD",
      "contractAddress": "0x84b06a41095be5536b3e6db1ee641ebc2f38cfcb",
      "networkId": 3
    }
  },
```

### Badges and Levels
You can enable user-badges for your tournament by enabling this feature. It will add a custom icon next to the users providers in the header, based on the amount of predictions they made.

```js
  "badges": {
    "enabled": true,
    "ranks": [
      {
        "icon": "assets/img/badges/junior-predictor.svg",
        "rank": "Junior Predictor",
        "minPredictions": 0,
        "maxPredictions": 4
      },
      {
        "icon": "assets/img/badges/crystal-gazer.svg",
        "rank": "Crystal Gazer",
        "minPredictions": 5,
        "maxPredictions": 9
      },
      {
        "icon": "assets/img/badges/fortune-teller.svg",
        "rank": "Fortune Teller",
        "minPredictions": 10,
        "maxPredictions": 14
      },
      {
        "icon": "assets/img/badges/clairvoyant.svg",
        "rank": "Clairvoyant",
        "minPredictions": 15,
        "maxPredictions": 19
      },
      {
        "icon": "assets/img/badges/psychic.svg",
        "rank": "Psychic",
        "minPredictions": 20
      }
    ]
  }
```

### **WIP**: Thirdparty Services
In order to determine which thirdparty integrations we want to use, we developed a plug-in system for integrations that can be included at a global scope, such as Google Analytics and the Chat Platform Intercom. To see how this was done, [take a look at the code](https://github.com/gnosis/pm-trading-ui/tree/master/src/utils/analytics).
```js
  "thirdparty": {
    "googleAnalytics": {
      "enabled": false,
      "config": {
        "id": "UA-000000-2"
      }
    },
    "intercom": {
      "enabled": false,
      "config": {
        "id": "INTERCOM_USERID"
      }
    }
  },
}
```

### Misc Constants
These are configurable constants in the application.

`LIMIT_MARGIN` - during trading it can happen that the margin for trade has been reduced by another users trade, after the specified amount (in percent) the user will receive a warning that the trade has been chaged.

`NOTIFICATION_TIMEOUT` - how long it takes for `uport` style notifications to be considered timed out.

`LOWEST_VALUE` - lowest possible value to display in the interface. Any value below will be shown `<${value}`, e.g. `Sell Price: <0.001`


```js
  "constants": {
    "LIMIT_MARGIN": 5,
    "NOTIFICATION_TIMEOUT": 60000,
    "LOWEST_VALUE": 0.001
  }
```
