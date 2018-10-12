# PM-JS
The pm-js library offers a convenient way of accesing the Gnosis contracts for prediction markets with Javascript and Node.js. We recommend the use of [pm-scripts](/pm-scripts) for the creation of markets and pm-js for dealing with the automated market maker functions: *BUY and SELL shares**. For interacting with the oracle contracts you might want to use it directly but pm-js allows to do it with some validation layers that are useful.

The use of pm-js assumes that you have a basic understanding of [Web3.js](https://github.com/ethereum/wiki/wiki/JavaScript-API) interface. It also uses [IPFS](https://ipfs.io/) for publishing and retrieving event data, and so it will also have to be connected to an IPFS node. 

## Install
Install [`pm-contracts`](https://github.com/gnosis/pm-contracts) and `pm-js` into your project as a dependency using:
   
       npm install --save '@gnosis.pm/pm-contracts' '@gnosis.pm/pm-js'
   
   Be sure to issue this command with this exact spelling. The quotes are there in case you use [Powershell](https://stackoverflow.com/a/5571703/1796894).

   This command installs the Gnosis core contracts and the Gnosis JavaScript library, and their dependencies into the `node_modules` directory. The [`@gnosis.pm/pm-js`](https://www.npmjs.com/package/@gnosis.pm/pm-js) package contains the following:

   * ES6 source of the library in `src` which can also be found on the [repository](https://github.com/gnosis/pm-js)
   * Compiled versions of the modules which can be run on Node.js in the `dist` directory
   * Webpacked standalone `gnosis-pm[.min].js` files ready for use by web clients in the `dist` directory
   * API documentation in the `docs` directory


Notice that the library refers to the `dist/index` module as the `package.json` main. This is because even though Node.js does support many new JavaScript features, ES6 import support is still very much in development yet (watch [this page](https://nodejs.org/api/esm.html#esm_ecmascript_modules)), so the modules are transpiled with [Babel](https://babeljs.io/) for Node interoperability.

In the project directory, you can experiment with the Gnosis API by opening up a `node` shell and importing the library like so:

```js
const Gnosis = require('@gnosis.pm/pm-js')
```

This will import the transpiled library through the `dist/index` entry point, which exports the [Gnosis](api-reference.html#Gnosis) class.

If you are playing around with pm-js directly in its project folder, you can import it from dist

```js
const Gnosis = require('.')
```

## Browser use

The `gnosis-pm.js` file and its minified version `gnosis-pm.min.js` are self-contained and can be used directly in a webpage. For example, you may copy `gnosis-pm.min.js` into a folder or onto your server, and in an HTML page, use the following code to import the library:

```html
<script src="gnosis-pm.min.js"></script>
<script>
// Gnosis should be available as a global after the above script import, so this subsequent script tag can make use of the API.
</script>
```

After opening the page, the browser console can also be used to experiment with the API.


