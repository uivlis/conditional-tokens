# pm-trading-db

Ethereum blockchain providers expose a common interface for querying, but to query blockchain providers directly for tournament-related info would be much too slow. That's why we've developed [pm-trading-db](https://github.com/gnosis/pm-trading-db), which syncs up with and queries a blockchain provider directly, filtering related data into an indexed database which provides great boosts in speed when querying the database.

Follow the pm-trading-db [**Tournament Setup** instructions](https://github.com/gnosis/pm-trading-db#tournament-setup). You will have a `geth --rinkeby` node running alongside an instance of pm-trading-db.

To cut down on the sync time, note that we don't actually have to start syncing the database until the first block in which a contract we are considering has been deployed. Let's take [BigToken](https://rinkeby.etherscan.io/address/0xd3515609e3231d6c5b049a28d0d09d038b4cfaed) for example again. Its [contract creation transaction](https://rinkeby.etherscan.io/tx/0xaa10a3d8ba2a08ae277eaadd5b876753ac118ede542ae89c25c882eda3766c53) occurred at a block height of [2105737](https://rinkeby.etherscan.io/block/2105737). This means the `setup_tournament` command could be invoked like this:

```sh
python manage.py setup_tournament --start-block-number 2105737
```
