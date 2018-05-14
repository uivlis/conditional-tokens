# Setting Up the Interface

First, [clone and setup the interface](https://github.com/gnosis/pm-trading-ui).

Refer to the [wiki](https://github.com/gnosis/pm-trading-ui/wiki/Configuration) for information about configuring the interface. First, you will want to create a folder for your tournament configuration. For this guide, let's assume a `big-tournament` folder has been created in `config/environments`.

Then, create a `config.json` in your newly created folder as described in the wiki:

```json
{
  "pm-trading-db": {
    "protocol": "http",
    "host": "localhost",
    "port": 8000
  },
  "ethereum": {
    "protocol": "https",
    "host": "rinkeby.infura.io",
    "port": 443
  },
  "whitelist": {
    "0x873faa4cddd5b157e8e5a57e7a5479afc5d30f0b": "Your name"
  }
}
```

Here, we use the `pm-trading-db` instance running on `localhost`, but if you have a location on the internet where your instance of pm-trading-db is reachable, you may specify that here. Also, be sure to whitelist your account. The whitelisted accounts will be queried for markets against pm-trading-db, and only those markets created by these accounts will be displayed by the interface.

You will also want to create an `interface.config.json`. You can follow the [wiki](https://github.com/gnosis/pm-trading-ui/wiki/Configuration#interface-config) to do a full configuration. Make sure the `tournament.registration.contractAddress` key is set to the address of the `AddressRegistry`, the `collateralToken.address` is set to the address of your `PlayToken`, and the `providers.default` is set to `METAMASK`:

```json
{
  ...,

  "tournament": {
    "enabled": true,
    "name": "Big Tournament",
    "registration": {
      "enabled": true,
      "contractAddress": "0xd3515609e3231d6c5b049a28d0d09d038b4cfaed"
    }
  },
  "collateralToken": {
    "address": "0x0152b7ed5a169e0292525fb2bf67ef1274010c74",
    "icon": "/assets/img/icons/icon_metamask.svg"
  },
  "providers": {
    "default": "METAMASK",
    "options": {
      "UPORT": {
        "appName": "Tournament test",
        "clientId": "2ooNzwDvjgZTPswkrewUnxokRUH8KYpRNNL",
        "network": "rinkeby",
        "privateKey": "a42611a039cad931c41e072953cb7dcaaa6b2dfefeff6a459d9ce4281acf0c01",
        "useNotifications": false
      }
    }
  },

  ...
}
```

## Reward Claiming

In order to allow tournament recipients to claim a reward of some ERC20 token, an instance of the `RewardClaimHandler` may be deployed, and that information must be relayed to the frontend.

To do so, you will need an account with actual Ether. We will use [MyEtherWallet](https://www.myetherwallet.com/#contracts) to do this deployment. For the purpose of this guide, we will issue rewards on the Kovan network, though you will almost certainly wish to use the public main network.

First we must deploy the contract `RewardClaimHandler` contract with MEW. You can find the bytecode in the `@gnosis.pm/pm-apollo-contracts` project in the build artifact `build/contracts/RewardClaimHandler.json` under the `bytecode` key, but this is reproduced for your convenience below:

```
0x6060604052341561000f57600080fd5b604051602080610c1b83398101604052808051906020019091905050806000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555033600160006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555050610b5f806100bc6000396000f30060606040526004361061008e576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806319ec3ded14610093578063553eb4db146100e0578063570ca735146101835780638c9845b0146101d8578063a2fb117514610201578063b88a802f14610264578063e110342214610279578063f7c618c11461028e575b600080fd5b341561009e57600080fd5b6100ca600480803573ffffffffffffffffffffffffffffffffffffffff169060200190919050506102e3565b6040518082815260200191505060405180910390f35b34156100eb57600080fd5b610181600480803590602001908201803590602001908080602002602001604051908101604052809392919081815260200183836020028082843782019150505050505091908035906020019082018035906020019080806020026020016040519081016040528093929190818152602001838360200280828437820191505050505050919080359060200190919050506102fb565b005b341561018e57600080fd5b610196610572565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b34156101e357600080fd5b6101eb610598565b6040518082815260200191505060405180910390f35b341561020c57600080fd5b610222600480803590602001909190505061059e565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b341561026f57600080fd5b6102776105dd565b005b341561028457600080fd5b61028c610759565b005b341561029957600080fd5b6102a16109f0565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b60036020528060005260406000206000915090505481565b6000806000600280549050148015610314575060008551115b8015610321575083518551145b801561037a5750600160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16145b151561038557600080fd5b60009150600090505b84518110156104325783818151811015156103a557fe5b906020019060200201518201915083818151811015156103c157fe5b906020019060200201516003600087848151811015156103dd57fe5b9060200190602002015173ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002081905550808060010191505061038e565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff166323b872dd3330856040518463ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019350505050602060405180830381600087803b151561052957600080fd5b5af1151561053657600080fd5b50505060405180519050151561054b57600080fd5b8460029080519060200190610561929190610a15565b508242016004819055505050505050565b600160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b60045481565b6002818154811015156105ad57fe5b90600052602060002090016000915054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b600060028054905011801561070757506000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1663a9059cbb33600360003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020546040518363ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401808373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200182815260200192505050602060405180830381600087803b15156106ef57600080fd5b5af115156106fc57600080fd5b505050604051805190505b151561071257600080fd5b6000600360003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002081905550565b60008060006002805490501180156107be5750600160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16145b80156107cc57506004544210155b15156107d757600080fd5b60009150600090505b6002805490508110156108f7576003600060028381548110151561080057fe5b906000526020600020900160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020548201915060006003600060028481548110151561088057fe5b906000526020600020900160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000208190555080806001019150506107e0565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1663a9059cbb33846040518363ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401808373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200182815260200192505050602060405180830381600087803b15156109ba57600080fd5b5af115156109c757600080fd5b5050506040518051905015156109dc57600080fd5b60006002816109eb9190610a9f565b505050565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b828054828255906000526020600020908101928215610a8e579160200282015b82811115610a8d5782518260006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555091602001919060010190610a35565b5b509050610a9b9190610acb565b5090565b815481835581811511610ac657818360005260206000209182019101610ac59190610b0e565b5b505050565b610b0b91905b80821115610b0757600081816101000a81549073ffffffffffffffffffffffffffffffffffffffff021916905550600101610ad1565b5090565b90565b610b3091905b80821115610b2c576000816000905550600101610b14565b5090565b905600a165627a7a723058200652f29a057236c9369c2c944946437d8769c29d3fdc0ea79bae1a4cd504018f0029
```

Then add the reward token address at the end, padded left to 64 hex characters (32 bytes/256 bits/one EVM word). For example, if your reward token address existed on the network at `0x3552D381b89Dcb92c59d7a0F8fe93b1e3BBE1886`, then we would append this to the bytecode:

```
0000000000000000000000003552D381b89Dcb92c59d7a0F8fe93b1e3BBE1886
```

Then deploy the contract. Once your contract is deployed, you should have an address. Let's say that address is `0x9720939c16665529dEaBE608bC3cA72509297F79`

Then, specify the `RewardClaimHandler` address along with additional parameters as `rewards.claimReward.contractAddress` in `config/interface.config.json` (note we are using Kovan's networkId in this example):

```json
{
  "rewards": {
    "enabled": true,
    "rewardToken": {
      "symbol": "RWD",
      "contractAddress": "0x3552D381b89Dcb92c59d7a0F8fe93b1e3BBE1886",
      "networkId": 42
    },
    "claimReward": {
      "claimStart": "2018-04-01T12:00:00",
      "claimUntil": "2018-05-01T12:00:00",
      "contractAddress": "0x9720939c16665529dEaBE608bC3cA72509297F79",
      "networkId": 42
    },
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
  },
  ...
}
```

Be aware that the `levels` key will show up on the scoreboard and signal to the players what their anticipated reward should be. Be sure that you have enough of the reward to offer!

### Post-tournament Reward Handling

When the tournament is over, you may use MEW to distribute the rewards to the players. To do so, first you must [approve](https://theethereum.wiki/w/index.php/ERC20_Token_Standard#Approve_And_TransferFrom_Token_Balance) the `RewardClaimHandler` the total amount of the reward token to send as a reward, as the `RewardClaimHandler` will attempt to `transferFrom` you that amount.

Afterwards, you may interact with your contract, using the `RewardClaimHandler` ABI which can also be found in the build artifact mentioned earlier in this section. Here is the ABI copied for convenience:

```json
[
  {
    "constant": true,
    "inputs": [
      {
        "name": "",
        "type": "address"
      }
    ],
    "name": "rewardAmounts",
    "outputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [],
    "name": "operator",
    "outputs": [
      {
        "name": "",
        "type": "address"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [],
    "name": "guaranteedClaimEndTime",
    "outputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "name": "winners",
    "outputs": [
      {
        "name": "",
        "type": "address"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [],
    "name": "rewardToken",
    "outputs": [
      {
        "name": "",
        "type": "address"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "name": "_rewardToken",
        "type": "address"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "constructor"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_winners",
        "type": "address[]"
      },
      {
        "name": "_rewardAmounts",
        "type": "uint256[]"
      },
      {
        "name": "duration",
        "type": "uint256"
      }
    ],
    "name": "registerRewards",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [],
    "name": "claimReward",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [],
    "name": "retractRewards",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```

You must call `registerRewards`. When calling that function, specify a list of the tournament winners' **mainnet addresses** (or in this example case, Kovan addresses), and their corresponding winning amounts, as well as a duration in seconds for claiming the reward. This duration should match the tournament configuration.

## Running the Dev Server

If you would like to run the webpack development server included with the project using this configuration, then use:

```sh
GNOSIS_ENV='big-tournament' npm start
```

(with `big-tournament` swapped out with whatever your choice is)

To produce a build of the frontend, use:

```sh
GNOSIS_ENV='big-tournament' npm build
```
