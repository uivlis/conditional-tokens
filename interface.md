# Setting Up the Interface

First, [clone and setup the interface](https://github.com/gnosis/gnosis-management).

Refer to the [wiki](https://github.com/gnosis/gnosis-management/wiki) for information about configuring the interface. In particular, make sure the following configuration variables are set accordingly:

```json
{
  "interface": {
    "tournament": true,
    "tournamentName": "My tournament",
    "tokenContract": "0x0152b7ed5a169e0292525fb2bf67ef1274010c74",
    "registrationContract": "0xd3515609e3231d6c5b049a28d0d09d038b4cfaed"
  }
}
```

Also, make sure that the whitelists are set in such a way that your tournament administrator accounts are whitelisted.

As we are developing, we might not have an instance of GnosisDB up in the cloud. If that is the case, just use the instance on your local machine:

```json
  "gnosisdb": {
    "protocol": "http",
    "hostDev": "localhost",
    "hostProd": "localhost",
    "port": 8000
  },
```
