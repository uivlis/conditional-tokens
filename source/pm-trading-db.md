# Trading DB Quickstart

Ethereum blockchain providers expose a common interface for querying, but to query blockchain providers directly for tournament-related info would be much too slow. That's why we've developed [pm-trading-db](https://github.com/gnosis/pm-trading-db), which syncs up with and queries a blockchain provider directly, filtering related data into an indexed database which provides great boosts in speed when querying the database.

## Tournament Setup

### Deploy your contracts
To configure a custom tournament, you need first to [deploy the smart contracts needed on the chosen public network](smart-contracts.md). For this setup guide, we will assume the choice of [Rinkeby](https://www.rinkeby.io/#stats).

Take note of your deployed addresses for [AddressRegistry](https://github.com/gnosis/pm-apollo-contracts#addressregistry) and the [PlayToken](https://github.com/gnosis/pm-apollo-contracts#playtoken). You can find them with `npm run truffle networks`. This guide will assume the following as the deployed addresses, though you will have something different:

```
OlympiaToken: 0x2924e2338356c912634a513150e6ff5be890f7a0
AddressRegistry: 0x12f73864dc1f603b2e62a36b210c294fd286f9fc
```

### Run ethereum node
You may have a running Geth node connected to [Rinkeby](https://www.rinkeby.io/#geth) on the same machine:

```sh
geth --rinkeby --rpc
```

If you don't want to have your own node you can use Infura's, but it will be slower than using a local node.

### Configure pm-trading-db
Clone the `pm-trading-db` repository:

```sh
git clone https://github.com/gnosis/pm-trading-db.git
cd pm-trading-db
```

Change these lines with the previous values in **.env_rinkeby**:

```sh
TOURNAMENT_TOKEN=0x2924e2338356c912634a513150e6ff5be890f7a0
GENERIC_IDENTITY_MANAGER_ADDRESS=0x12f73864dc1f603b2e62a36b210c294fd286f9fc
```

Add your ethereum account too for **token issuance**. You will need some **ether** on it for gas costs:

```sh
ETHEREUM_DEFAULT_ACCOUNT=0x847968C6407F32eb261dC19c3C558C445931C9fF
ETHEREUM_DEFAULT_ACCOUNT_PRIVATE_KEY=a3b12a165350ab3c7d1ecd3596096969db2839c7899a3b0b39dd479fdd5148c7
```

If you don't have the private key for your account, but you do know the [BIP39](https://iancoleman.io/bip39/) mnemonic for it, you may enter your mnemonic into [Ganache](https://truffleframework.com/ganache) to recover the private key. You can omit the `ETHEREUM_DEFAULT_ACCOUNT_PRIVATE_KEY` if you have the **address unlocked** in your Ethereum node.

Configure the HTTP provider on **.env** and change `DJANGO_SETTINGS_MODULE=config.settings.local` to `DJANGO_SETTINGS_MODULE=config.settings.rinkeby`:

```
DJANGO_SETTINGS_MODULE=config.settings.rinkeby
ETHEREUM_NODE_URL=http://172.17.0.1:8545
```

We wrote **172.17.0.1** because that's usually the IP of the **docker host**, and **geth** is running in your machine, not in docker.

Then in **pm-trading-db root folder**:

### Run pm-trading-db in Docker
```
docker-compose build --force-rm
docker-compose run web sh
python manage.py migrate
python manage.py setup_tournament --start-block-number 2000000
exit
docker-compose up
```

The command `setup_tournament` will prepare the database and set up periodic tasks:
  - `--start-block-number` will, if specified, start pm-trading-db processing at a specific block instead of all the way back at the genesis block. You should give it as late a block before tournament events start occurring as you can. We don't actually have to start syncing the database until the first block in which a contract we are considering has been deployed. Let's take [BigToken](https://rinkeby.etherscan.io/address/0xd3515609e3231d6c5b049a28d0d09d038b4cfaed) for example again. Its [contract creation transaction](https://rinkeby.etherscan.io/tx/0xaa10a3d8ba2a08ae277eaadd5b876753ac118ede542ae89c25c882eda3766c53) occurred at a block height of [2105737](https://rinkeby.etherscan.io/block/2105737). This means the `setup_tournament` command could be invoked like this:
    ```sh
    python manage.py setup_tournament --start-block-number 2105737
    ```
  - **Ethereum blockchain event listener** every 5 seconds (the main task of the application).
  - **Scoreboard calculation** every 10 minutes.
  - **Token issuance** every minute. Tokens will be issued in batches of 50 users (to prevent
  exceeding the block limitation). A flag will be set to prevent users from being issued again on next
  execution of the task.
  - **Token issuance flag clear**. Once a day the token issuance flag will be cleared so users will
  receive new tokens every day.

### Final steps
All these tasks can be changed in the [application admin](http://localhost:8000/admin/django_celery_beat/periodictask/).
You will need a superuser:

```
docker-compose run web sh
python manage.py createsuperuser
```

You should have now the api running in http://localhost:8000. You have to be patient because the
first synchronization of Rinkeby may take some time, depending on how many blocks pm-trading-db has to process. It may take even more time if your Geth node is unsynchronized, since it [may need to finish synchronizing](https://github.com/ethereum/go-ethereum/issues/14338) before it will have the information required.

At this point you should have a `geth --rinkeby` node running alongside an instance of **pm-trading-db**.

## Custom event receivers

### Code Python event receiver
Custom event receivers can be set up in Tradingdb extending `django_eth_events.chainevents.AbstractEventReceiver` and then define methods:
  - `save(decoded_event, block_info)`: Will process events when received. `block_info` will return [web3 block structure](https://web3py.readthedocs.io/en/stable/web3.eth.html#web3.eth.Eth.getBlock) of the ethereum block where the event is found.
  - `rollback(decoded_event, block_info)`: Will process events in case of reorg. The event will be the same that in `save`, so you decide how to rollback the changes (in case that's needed).

You will be able to listen for events on your own contracts.

Every `decoded_event` has `address` and `name`, and then decoded params under `params` key. `address` are always lowercase without `0x`. Example of event:
```js
{
    "address": "b3289eaac0fe3ed15df177f925c6f8ceeb908b8f",
    "name": "CentralizedOracleCreation",
    "params": [
        {
            "name": "creator",
            "value": "67ed2c5c09b7aa308dbd0fb8754b695e5bb030ad"
        },
        {
            "name": "centralizedOracle",
            "value": "88c2c1bb33c4939f58384629e7b5f26d90bafcc9"
        },
        {
            "name": "ipfsHash",
            "value": "QmNUhQD2hzRb8Pj31RHtBaJNpZUzQ9cg1AKKW8SFVScFb5"
        }
    ]
}
```

You can add a custom **EventReceiver** to the event receivers file **tradingdb/chainevents/event_receivers.py**. An example of EventReceiver:
```python
from django_eth_events.chainevents import AbstractEventReceiver

class TestEventReceiver(AbstractEventReceiver):
    def save(self, decoded_event, block_info=None):
        event_name = decoded_event.get('name')
        address = decoded_event.get('address')

        print('Received event', event_name, 'with address', address)
        print(decoded_event.get('params'))

    def rollback(self, decoded_event, block_info=None):
        # Undo stuff done by `save` in case of reorg
        # For example, delete a database object created on `save`
        # No need for rollback in this case
        pass
```

### Add contract ABI
If you want to listen events for your **own contract**, you need to add the **json ABI** to **tradingdb/chainevents/abis/** folder.

Then you need to configure your listener before starting **pm-tradingdb** for the first time. Go to **config/settings/olympia.py** and add your event listener as a Python dictionary. Required fields are:
  - **ADDRESSES**: Address of the contract/s to the events to be listened.
  - **EVENT_ABI**: ABI of your custom contract (used to decode the events).
  - **EVENT_DATA_RECEIVER**: Absolute python import path for the custom event listener class.
  - **NAME**: Name of the receiver, just don't use same name that another receiver.


### Configure event listener

Example of a custom event listener:
```js
{
    'ADDRESSES': ['0xD6fF69322719b077fDC5335535989Aa702016276', '0x992575d97fa3C31f39a81BDC3D517aE7D8C1C5A2'],
    'EVENT_ABI': load_json_file(abi_file_path('MyTestContract.json')),
    'EVENT_DATA_RECEIVER': 'chainevents.event_receivers.TournamentTokenReceiver',
    'NAME': 'OlympiaToken',
},
```

You should now be ready to run **pm-trading-db**