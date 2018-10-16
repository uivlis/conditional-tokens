# Trading DB

We described in a [previous section](prediction-markets-as-modular-framework.html#ethereum-indexer) the benefits of having an ethereum indexer over the classical approach of quering directly the blockchain.
In this section we will cover the differnt ways of executing it, configurations and more advanced scennarios where you will need to modify our indexer to fit with your custom smart contract modules and all the different configuration parameters available.

TradingDB is a python project based on [django](https://www.djangoproject.com/) and [celery](http://www.celeryproject.org/) that follows an architecture of micro-services, with 3 main components: the web API, a scheduler (producer) and worker (consumer). These 3 componentes use a message queu to communicate, by default we use Redis and also a Relational database (we recommend postgresql).
There are other external services needed as and ethereum node (depending on the use case you could use infura) and an IPFS node (you can use infura for this).

There are many ways of executing this software, but mainly those are 4:
* [Docker-compose](#docker-compose)
* Docker
* Container Orchestrator. e.g Kubernetes
* Bare-metal 

## Docker-compose
[docker-compose](https://docs.docker.com/compose/) is a a tool for defining and running multi-container Docker applications and link the dependencies between them. You can see it as tool for managing micro-service and container projects as monoliths, to make the execution easier for development.

If you take a look at our [docker-compose.yml](https://github.com/gnosis/pm-trading-db/blob/master/docker-compose.yml) we have defined many services inside, like the postgresql database and the redis cache. This makes the onboarding very easy, everything you need to do is running two commands.
```
docker-compose build
docker-compose up
```

The `up` command will run forever. In case you need to access one of the services for management, you can use `run`:
```
docker-compose run web sh
docker-compose run worker sh
docker-compose run scheduler sh
python manage.py
``` 

Note that by default, the configuration used is for the Rinkeby network. Check `config.settings.rinkeby`

## Docker
You can check the tradingdb images in the public docker registry [here](https://hub.docker.com/r/gnosispm/pm-trading-db/tags/). The same image can be used for the 3 pieces of the system: web, scheduler and worker.

Basically you will need to run a different command for each piece:
* Web: `docker/web/run_web.sh` [code](https://github.com/gnosis/pm-trading-db/blob/v1.7.3/docker/web/run_web.sh)
* Scheduler: `docker/web/celery/scheduler/run.sh` [code](https://github.com/gnosis/pm-trading-db/blob/v1.7.3/docker/web/celery/scheduler/run.sh)
* Worker: `docker/web/celery/worker/run.sh` [code](https://github.com/gnosis/pm-trading-db/blob/v1.7.3/docker/web/celery/worker/run.sh)

Running it directly with docker will mean you need to manage restarts, failures and connections. Take a look at the configuration section to know which parameters do you need to pass from environment.

## Kubernetes
[Kubernetes](https://kubernetes.io/) is one of the most robust solutions for container orchestration and is what **we recommend for production** of TradingDB.

In order to run this project on your kubernetes cluster, you need to follow these steps:

```sh
# Verify Kubernetes version is > 1.9
kubectl get nodes
```

### Database configuration
The database configuration of tradingdb uses [kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/) for storing this sensitive information.
```sh
kubectl create secret generic tradingdb-database \
--from-literal host='[DATABASE_HOST]' \
--from-literal name=[DATABASE_NAME] \
--from-literal user=[DATABASE_USER] \
--from-literal password='[DATABASE_PASSWORD]' \
--from-literal port=[DATABASE_PORT]
```

### Queue and Cache
We use [Redis](https://redis.io/) as message broker for Celery (handles the different tasks messages as indexing, issuing tokens, etc) and also as the cache service.
You can apply it in your cluster by:
```
kubectl apply -f kubernetes/redis-tradingdb
```
By default this creates a deployment and a service in kubernetes. You should not need further configuration for this part.

### TradingdDB services
As we explained in the previous section, tradingdDB follows a microservice architecture, and it's core is formed by 3 services: `worker, scheduler and web API`. In order to deploy these services there are a minimum of configuration parameters you need to set up like:
* Ethereum node URL (by default points to infura, in production you should have an ethereum node that supports many requests per second). Besides what you would think, the interface with better performance is the RPC API (over Webservices or IPC sockets). This is because we can batch requests in the same HTTP connection and the underliying implementation of ethereum nodes it's more efficient for the RPC API.
* DJANGO_SECRET_KEY this parameter secures your sessions with the admin interface (/admin). In a UNIX environment you can generate a random string with this command: `head /dev/urandom | shasum -a 512`

There are many more parameters we describe in the [configuration section](/tradingdb-configuration).

After you have your configuration set, apply the deployment with:
```sh
kubectl apply -f kubernetes/tradingdb
```

## Bare metal
TradingDB it's a **Python 3.6/Django 2** project, so you can set up the application without *Docker*.

You will need installed an running:
* Redis: Tasks management.
* PostgreSQL: As the database. Create a database for the project.

Then you will need **Python 3.6**, and it's recommended to have [virtualenv](https://virtualenv.pypa.io/en/stable/) for the dependencies:
```bash
git clone git@github.com:gnosis/pm-trading-db.git
cd pm-trading-db
virtualenv pm-trading-db
pip install -r requirements.txt
```

Configure `.env_bare_local` with parameters for connecting to PostgreSQL and Redis. You can use Infura for Ethereum node and IPFS:
```bash
DATABASE_URL=psql://user:password@localhost:5432/database_name
REDIS_URL=redis://localhost/0
CELERY_BROKER_URL=redis://localhost/0
ETHEREUM_NODE_URL=https://rinkeby.infura.io/YOUR_INFURA_TOKEN
IPFS_HOST=https://ipfs.infura.io
IPFS_PORT=5001
```

Then run:
```bash
python manage.py migrate
python manage.py setup_tournament --start-block-number 2000000
./run_bare_metal.sh
```

Tradingdb should be up and running on [http://0.0.0.0:8000](http://0.0.0.0:8000)

# Configuration Parameters
In the project you will find some configuration templates for different environments. These are in `config/settings/`:
* `base.py` As the name says, it's the base of all the config parameters, has the common configurations and the default values.
* `ganache.py` You should use this config when testing with [ganache-cli](https://github.com/trufflesuite/ganache-cli) running `ganache-cli -d`.
* `production.py` Disables the debug settings and is oriented to be use on mainnet (or a testnet for running an olympia tournament).
* `rinkeby.py` Has configured the default addresses for rinkeby and also for one of the Olympia tournaments [Gnosis run](https://blog.gnosis.pm/announcing-gnosis-olympia-dappcon-edition-be44643a046e), as an example.
* `test.py` Used by tests.

Here you have a list of all the possible parameters you can set as ENV parameter (not all configs allows to override by ENV).

### DJANGO_DEBUG
`bool` - Enables debug logs. Makes it easier for finding bugs in the API.
### DATABASE_URL
`url` - Database url used by the service, follows [django-environ supported db_url](https://github.com/joke2k/django-environ#supported-types)
### CELERY_BROKER_URL
`url` - Follows [this format](https://kombu.readthedocs.io/en/master/userguide/connections.html#connection-urls)

### ETH_BACKUP_BLOCKS 
`int` - amount of blocks saved for rollbacks (chain reorgs). It's 100 by default.

### ETH_PROCESS_BLOCKS 
`int` - number of blocks processed as bulk for the indexer every time an indexing task is triggered (by default every 500ms). Increasing this value will mean "maybe" the indexing will be faster, but that will also depend on the cpu, memory and network resources. There will be many RPC requests and you might kill you ethereum node instance ^^.

### ETH_FILTER_MAX_BLOCKS 
`int` - follows the same concept than the previous parameter but with the difference that instead of performing pulling of ethereum logs, it uses ethereum filters. Ethereum filters are used for the first sync as those are faster for synchronizing historic data.

### ETHEREUM_NODE_URL (mandatory in production)
`protocol://host:port` - The RPC endpoint of your ethereum node.

### ETHEREUM_MAX_WORKERS 
`int` - default `10`. Represents the amount of parallel processes performing requests to the ethereum node.

### ETHEREUM_MAX_BATCH_REQUESTS 
`int` - default `500`. Amount of RPC requests batched in one single HTTP request.

### IPFS_HOST
`string` - default `ipfs.infura.io`

### IPFS_PORT
`int` - default `5001`

### ALLOWED_HOSTS
`string` - Separated by commas, url and ips allowed to be used for the API.

### LMSR_MARKET_MAKER
`ethereum checksum address` - Automated market maker allowed to be used by market contracts. You can check the default addresses for each network [here](https://unpkg.com/@gnosis.pm/pm-contracts@1.1.0/networks.json)

### CENTRALIZED_ORACLE_FACTORY
`ethereum checksum address` - Centralized Oracle factory contract. You can check the default addresses for each network [here](https://unpkg.com/@gnosis.pm/pm-contracts@1.1.0/networks.json)

### EVENT_FACTORY
`ethereum checksum address` - Event factory contract. You can check the default addresses for each network [here](https://unpkg.com/@gnosis.pm/pm-contracts@1.1.0/networks.json)

### MARKET_FACTORY
`ethereum checksum address` - Market factory contract. You can check the default addresses for each network [here](https://unpkg.com/@gnosis.pm/pm-contracts@1.1.0/networks.json)

### GENERIC_IDENTITY_MANAGER_ADDRESS (Olympia related)
`ethereum checksum address` - Registry address contract, you need to deploy it by yourself. Will be the registry for new users of a tournament.

### TOURNAMENT_TOKEN (Olympia related)
`ethereum checksum address` - Represents the [Olympia token](https://github.com/gnosis/pm-apollo-contracts/blob/master/contracts/OlympiaToken.sol) used for tournament (use 0x0000000000000000000000000000000000000001 if you want to disable it)

### ETHEREUM_DEFAULT_ACCOUNT_PRIVATE_KEY (Olympia related)
`string` - Ethereum private key used for tournament tokens issuance. This account should be the creator of the tournament token.

### TOURNAMENT_TOKEN_ISSUANCE (Olympia related)
`int` - Amount of tokens to be issued per participant. **NOTE THE AMOUNT IS IN WEI UNITS**

### ISSUANCE_GAS (Olympia related)
`int` - Gas limit of issuance transactions.

### ISSUANCE_GAS_PRICE (Olympia related)
`int` - Gas price used for issuance transactions.


# Extend TradingDB (ADVANCED)
There are many reason why you would like to extend the project, the main one is that you have custom contracts and specific data that you would like to save in the indexer or trigger some actions (like send an email after a deposit transfer).

## Implement Python event receiver
With custom event receivers you will be able to listen for events on your own contracts. Custom event receivers can be set up in **pm-trading-db** extending `django_eth_events.chainevents.AbstractEventReceiver` and then defining methods:
  - `save(decoded_event, block_info)`: Will process events when received. `block_info` will have the [web3 block structure](https://web3py.readthedocs.io/en/stable/web3.eth.html#web3.eth.Eth.getBlock) of the ethereum block where the event is found.
  - `rollback(decoded_event, block_info)`: Will process events in case of reorg. The event will be the same that in `save`, so you decide how to rollback the changes (in case that's needed).

Every `decoded_event` has `address` and `name`, and then decoded params under `params` key. `address` is always lowercase without `0x`. Example of event:
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

## Add contract ABI
If you want to listen events for your **own contract**, you need to add the **json ABI** to **tradingdb/chainevents/abis/** folder to make pm-trading-db capable of decoding the events.

Then you need to configure your receiver before starting **pm-tradingdb** for the first time. Go to **config/settings/olympia.py** and add your event receiver as a Python dictionary. Required fields are:
  - **ADDRESSES**: List addresses of the contracts to be watched for events. If you need to watch one single address, use a one element list.
  - **EVENT_ABI**: ABI of your custom contract (used to decode the events).
  - **EVENT_DATA_RECEIVER**: Absolute python import path for the custom event receiver class.
  - **NAME**: Name of the receiver, just don't use same name that another receiver.


### Configure custom event receiver

Example of a custom event receiver:
```js
{
    'ADDRESSES': ['0xD6fF69322719b077fDC5335535989Aa702016276', '0x992575d97fa3C31f39a81BDC3D517aE7D8C1C5A2'],
    'EVENT_ABI': load_json_file(abi_file_path('MyTestContract.json')),
    'EVENT_DATA_RECEIVER': 'chainevents.event_receivers.TournamentTokenReceiver',
    'NAME': 'OlympiaToken',
},
```

You should now be ready to run **tradingDB**
