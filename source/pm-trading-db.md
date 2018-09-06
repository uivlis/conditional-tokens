# Trading DB Quickstart

Ethereum blockchain providers expose a common interface for querying, but to query blockchain providers directly for tournament-related info would be much too slow. That's why we've developed [pm-trading-db](https://github.com/gnosis/pm-trading-db), which syncs up with and queries a blockchain provider directly, filtering related data into an indexed database which provides great boosts in speed when querying the database.

## TOURNAMENT SETUP

To configure a custom tournament, you need first to [deploy the smart contracts needed on the chosen public test network](smart-contracts.md). For this setup guide, we will assume the choice of [Rinkeby](https://www.rinkeby.io/#stats).

Take note of your deployed addresses for [AddressRegistry](https://github.com/gnosis/pm-apollo-contracts#addressregistry) and the [PlayToken](https://github.com/gnosis/pm-apollo-contracts#playtoken). You can find them with `npm run truffle networks`. This guide will assume the following as the deployed addresses, though you will have something different:

```
OlympiaToken: 0x2924e2338356c912634a513150e6ff5be890f7a0
AddressRegistry: 0x12f73864dc1f603b2e62a36b210c294fd286f9fc
```

Clone the `pm-trading-db` repository:

```sh
git clone https://github.com/gnosis/pm-trading-db.git
cd pm-trading-db
```

Change these lines with your custom values in **.env_rinkeby**:

```sh
TOURNAMENT_TOKEN=0x2924e2338356c912634a513150e6ff5be890f7a0
GENERIC_IDENTITY_MANAGER_ADDRESS=0x12f73864dc1f603b2e62a36b210c294fd286f9fc
```

Add your ethereum account too for token issuance. You need some ether on it for gas costs:

```
ETHEREUM_DEFAULT_ACCOUNT=0x847968C6407F32eb261dC19c3C558C445931C9fF
ETHEREUM_DEFAULT_ACCOUNT_PRIVATE_KEY=a3b12a165350ab3c7d1ecd3596096969db2839c7899a3b0b39dd479fdd5148c7
```

If you don't have the private key for your account, but you do know the BIP39 mnemonic for it, you may enter your mnemonic into [Ganache](http://truffleframework.com/ganache/) to recover the private key. You can omit the `ETHEREUM_DEFAULT_ACCOUNT_PRIVATE_KEY` if you have the **address unlocked** in your Ethereum node.

You may have a running Geth node connected to [Rinkeby](https://www.rinkeby.io/#geth) on the same machine:

```sh
geth --rinkeby --rpc
```

Configure the HTTP provider on **.env** and change `DJANGO_SETTINGS_MODULE=config.settings.local` to `DJANGO_SETTINGS_MODULE=config.settings.rinkeby`:

```
DJANGO_SETTINGS_MODULE=config.settings.rinkeby
ETHEREUM_NODE_URL=http://172.17.0.1:8545
```

Then in **pm-trading-db root folder**:

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

All these tasks can be changed in the [application admin](http://localhost:8000/admin/django_celery_beat/periodictask/).
You will need a superuser:

```
docker-compose run web sh
python manage.py createsuperuser
```

You should have now the api running in http://localhost:8000. You have to be patient because the
first synchronization of Rinkeby may take some time, depending on how many blocks pm-trading-db has to process. It may take even more time if your Geth node is unsynchronized, since it [may need to finish synchronizing](https://github.com/ethereum/go-ethereum/issues/14338) before it will have the information required.

At this point you should have a `geth --rinkeby` node running alongside an instance of **pm-trading-db**.