*Note: We **highly** recommend viewing this tutorial in Light Mode. Steemit does not support dark themes for inline and code blocks, making this page difficult to read in Night Mode.*<br>

<hr>

In this tutorial series, we will be creating a dapp for Ethereum.  It will be broken into three parts: 
1.  [The Smart Contract](https://steemit.com/tutorial/@hardlydifficult/ethereum-dapp-tutorial-part-1-of-3-solidity-smart-contract)
2.  [Web front-end with Metamask integration](https://steemit.com/tutorial/@hardlydifficult/ethereum-dapp-tutorial-part-2-of-3-web-front-end-with-metamask-integration)
3.  **Ledger integration** (this part)

We’ll be creating a simple dapp called ‘Message of the Moment,’ which will display a message that anyone is welcome to change.

Full source code for Part 3 is at the bottom of the post.

<hr>

<h1>Part 3: Ledger Integration</h1>
This tutorial will utilize the following references and resources. Don’t worry about collecting them all right now; we’ll refer to them inline as we use them.

* [Infura](https://infura.io/)
* [LedgerJS](https://github.com/LedgerHQ/ledgerjs)
* [NPM and NodeJS](https://www.npmjs.com)
* [Webpack](https://webpack.js.org/guides/getting-started/)
* [Web3.js](https://github.com/ethereum/web3.js/)
* [jQuery](https://jquery.com/)

<br>

##  Setting Up the Environment 
In part 2 we used a minimal Webpack deployment for testing.  In this part we'll need a more capable environment for development.  

You may prefer to use a template such as [Vuejs's Webpack Template](https://github.com/vuejs-templates/webpack).  This makes setup easy and adds features like hot reloading, but steepens the learning curve.


### 1.  Run `npm init -y`
This command will create a `package.json` file, required for managing the packages we will be adding below.

#### `-y`

The `-y` param accepts all the defaults.  You can remove this and then answer a series of questions, configuring metadata for the application.

<br>

### 2.  Move the javascript to `index.js`
Remove all the javascript we wrote in `index.html` and add it to the `/src/index.js` file.  

We are moving the javascript in order to support `include` statements.  Templates such as the Vuejs template mentioned above can make this more seemless.

<br>

### 3.  Include `dist/main.js` in `index.html`

```html
<head>
  <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
  <script src="./dist/main.js"></script>
</head>
```

This file, `dist/main.js,` is a bundle of all your other js files, generated when we do a webpack build as described below.

<br>

### 4.  In the button, replace the event with an id
Since we separated the js into another file, we will attach the javascript event handler to the button instead of the other way around.  In other words, the button does nothing until the javascript file has been initialized.

Replace:

```html
  <button onclick="setMessage()">
```

With:

```html
  <button id="set_message">
```

<br>

<h3>5.  Attach the button's event handler from `index.js`</h3>

Inside the callback for the '`load`' event, attach the `setMessage` function to the button named `set_message`.

We do this on `load` to ensure that the DOM has loaded, otherwise it may execute before the HTML is available... and then fail to attach.

```javascript
window.addEventListener('load', () => {
  ...
  $('#set_message').click(setMessage);
});
```

#### `.click(...)`
jQuery for setting a callback for responding to a click event.

<br>

### 6.  Launch the server 

Same as last time:

```
npx webpack-dev-server --https
```


<br>

### 7.  Build the application

Execute this command in a new command window so that you don't need to stop the server.

```
npx webpack --mode=development
```

This compiles your application, generating the `dist/main.js` required when viewing the site.

This command needs to be repeated anytime you change a js file.  There are hot-reloading options available to make development easier (included with the Vuejs template mentioned above). 

#### `--mode=development`

This option prevents things like compression in order to make debugging a bit easier.

### 8.  Test

At this point, the app should work exactly as it did at the end of Part 2.  It should work so long as the user has Metamask installed and the correct network selected.

<h2>Update Web3 and Add Ledger Support</h2>

### 9. Install Web3.js and the Ledger components

```
npm install web3 web3-provider-engine @ledgerhq/web3-subprovider @ledgerhq/hw-transport-u2f
```

#### `web3`

We need to install Web3.js to use if Metamask is not available. 

#### `web3-provider-engine`

This allows us to use Web3.js with custom providers.  We use this to enable two features: 1) Ledger integration and 2) read-only support when neither Metamask nor Ledger is available.

#### `@ledgerhq/web3-subprovider  @ledgerhq/hw-transport-u2f`

These two components are required to interface with the Ledger Nano S hardware wallet.

<br>

### 10.  Import Web3.js

Add the following to the top of `index.js`:
 
```javascript
import Web3 from "web3";
```

<br>

### 11.  Declare global variables

```javascript
let my_web3;
let account;
const rpcUrl = "https://ropsten.infura.io";
```

#### `my_web3`

Metamask does not use the latest version of Web3.js, presumably to maintain backwards compatibility for dapps.  In order to have consistent code when working with either Metamask or Ledger, we will be replacing the `web3` object Metamask provides with this `my_web3` object.

#### `account`

By pulling in the Web3.js library, we are able to support read-only calls for users which don't have either Metamask or Ledger connected (i.e. anyone can read the current message).  

This variable allows us to block any attempt to create a transaction (which would fail anyways).

#### `rpcUrl`

Our application is reading information from an Ethereum node.  Metamask will select the node automatically, but we need to do it ourselves for users without Metamask.

We are using [Infura](https://infura.io/) and the Ropsten testnet.


<br>

### 12.  Construct the `my_web3` object

Remove the 'Metamask is not installed' check, it's no longer an error as we can support users without Metamask now.  Replace it with the following:

```javascript
window.addEventListener('load', () => {
  if(typeof(web3) === 'undefined') {
    my_web3 = new Web3(new Web3.providers.HttpProvider(rpcUrl));
  } else {
    my_web3 = new Web3(web3.currentProvider);    
  }

  ...
}
```


#### `Web3.providers.HttpProvider`

This defines a node for the API to connect to for making requests.  This enables read-only support for anyone viewing the site, even if they do not have an Ethereum account.

#### `web3.currentProvider`

This allows the latest version of Web3.js to the Metamask provider, allowing the application to interface with `my_web3` consistently (i.e. the same code will work for Metamask and for Ledger users).

<br>

### 13.  Update the Web3.js calls to the latest standard

```
  contract = new my_web3.eth.Contract(abi, contract_address);
  contract.methods.message().call((error, result) => {
    ...
  }).catch((error) => {
    console.log("Error: " + error);
  });

  ...

  contract.methods.setMessage(message).send(
    {gasPrice: my_web3.utils.toWei("4.1", 'Gwei')},
    ...
  ).catch((error) => {
    console.log("Error: " + error);
  });
```

#### `my_web3`

We will be replacing all other instances of `web3` with `my_web3`, so that calls are the same for all users.

#### `eth.Contract`

With the newer version of Web3.js, constructing a contract has changed.  

#### `.methods`

The contract object moved all the data and methods under this `.methods` type.  Additionally when calling a method, the paramaters you are passing in are moved to the method itself vs inside `.call` or `.send`.

#### `.utils`

Helper methods such as .toWei have moved under .utils, but otherwise work the same.

#### `.catch`

The newer version of Web3.js uses Promises, if an error is thrown you may respond to it here.

### 14.  Test again, we added a feature but lost one as well

Rebuild!

```
npx webpack --mode=development
```

Anyone can now read the message.  Previously that only worked for users with Metamask installed.  The easiest way to test the experience without Metamask installed is by using a new incognito window.

You will not be able to change the message though.  If you try, with or without Metamask, it throws an error 'No "from" address specified in neither the given options, nor the default options.'

<br>

### 15.  Get the user's address

Add the following after setting the contract variable (`contract = new...`):

```javascript
my_web3.eth.getAccounts((error, result) => {
  if(error) {
    console.log(error);
  } else if(result.length == 0) {
    console.log("You are not logged in");
  } else {
    account = result[0];
    contract.options.from = account;
  }
}).catch((error) => {
  console.log("Error: " + error);
});
```

Rebuild and test, you can now change the message again.

<br>

### 16.  Add Ledger support

Import the following at the top of your `index.js` file:

```javascript
import createLedgerSubprovider from "@ledgerhq/web3-subprovider";
import TransportU2F from "@ledgerhq/hw-transport-u2f";
import ProviderEngine from "web3-provider-engine";
import RpcSubprovider from "web3-provider-engine/subproviders/rpc";
```

Inside the '`load`' event, add the following:

```javascript
window.addEventListener('load', () => {
  const use_ledger = location.search.indexOf("ledger=true") >= 0;

  if(use_ledger)
  {
    const engine = new ProviderEngine();
    const getTransport = () => TransportU2F.create();
    const ledger = createLedgerSubprovider(getTransport, {
      networkId: 3, // 3 == Ropsten testnet
    });
    engine.addProvider(ledger);
    engine.addProvider(new RpcSubprovider({ rpcUrl }));
    engine.start();
    my_web3 = new Web3(engine); 
  } else if(typeof(web3) === ...
```

#### `location.search.indexOf("ledger=true") >= 0`

A poor mans solution allowing the user to select when to use Ledger.  We'll implement the front-end for that next.

#### `ProviderEngine`

Provider engine's are a pattern used to connect an implementation to the Web3.js API.

#### `TransportU2F`

The hardware communication layer for the Ledger device.

#### `createLedgerSubprovider`

This handles communications with the device for authenticated actions, such as getting the user's account or sending a transaction.

#### `RpcSubprovider`

Rpc is added as well to enable read-only calls, which do not require communication with the Ledger hardware device.


<br>

### 17.  Create a widget to select Ledger

Add the following to your `index.html`:

```html
<a href="?ledger=false">Metamask</a> | <a href="?ledger=true">Ledger</a>
```

To test:

 - Rebuild!
 - Plug in your Ledger Nano S
 - Open the Ethereum app
 - Make sure both 'Contract data' and 'Browser support' are enabled.
 - Refresh the page and then attempt a transaction.

<hr>

<br>

That’s it!  Hope this was helpful.  

Next steps:
 - Web3.js's getTransactionReceipt function.
 - Use getNetwork wrong network selected in metamask.

<br>

<hr>

<h1>Source Code - Part 3</h1>

```html
import Web3 from "web3";
import createLedgerSubprovider from "@ledgerhq/web3-subprovider";
import TransportU2F from "@ledgerhq/hw-transport-u2f";
import ProviderEngine from "web3-provider-engine";
import RpcSubprovider from "web3-provider-engine/subproviders/rpc";

const contract_address = "0x654b54c945d29981d597fc8756cdb3c6e372440c";
const abi = [
{
    "anonymous": false,
    "inputs": [
        {
            "indexed": true,
            "name": "previousOwner",
            "type": "address"
        }
    ],
    "name": "OwnershipRenounced",
    "type": "event"
},
{
    "anonymous": false,
    "inputs": [
        {
            "indexed": true,
            "name": "previousOwner",
            "type": "address"
        },
        {
            "indexed": true,
            "name": "newOwner",
            "type": "address"
        }
    ],
    "name": "OwnershipTransferred",
    "type": "event"
},
{
    "constant": false,
    "inputs": [],
    "name": "renounceOwnership",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
},
{
    "constant": false,
    "inputs": [
        {
            "name": "_maxLength",
            "type": "uint256"
        }
    ],
    "name": "setMaxLength",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
},
{
    "constant": false,
    "inputs": [
        {
            "name": "_message",
            "type": "string"
        }
    ],
    "name": "setMessage",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
},
{
    "constant": false,
    "inputs": [
        {
            "name": "_newOwner",
            "type": "address"
        }
    ],
    "name": "transferOwnership",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
},
{
    "inputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "constructor"
},
{
    "constant": true,
    "inputs": [],
    "name": "maxLength",
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
    "name": "message",
    "outputs": [
        {
            "name": "",
            "type": "string"
        }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
},
{
    "constant": true,
    "inputs": [],
    "name": "owner",
    "outputs": [
        {
            "name": "",
            "type": "address"
        }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
}
];

let my_web3;
let account;
const rpcUrl = "https://ropsten.infura.io";
let contract;
window.addEventListener('load', () => {
  const use_ledger = location.search.indexOf("ledger=true") >= 0;

  if(use_ledger)
  {
    const engine = new ProviderEngine();
    const getTransport = () => TransportU2F.create();
    const ledger = createLedgerSubprovider(getTransport, {
      networkId: 3, // 3 == Ropsten testnet
    });
    engine.addProvider(ledger);
    engine.addProvider(new RpcSubprovider({ rpcUrl }));
    engine.start();
    my_web3 = new Web3(engine); 
  } else if(typeof(web3) === 'undefined') {
    my_web3 = new Web3(new Web3.providers.HttpProvider(rpcUrl));
  } else {
    my_web3 = new Web3(web3.currentProvider);    
  }
  contract = new my_web3.eth.Contract(abi, contract_address);
  my_web3.eth.getAccounts((error, result) => {
    if(error) {
      console.log(error);
    } else if(result.length == 0) {
      console.log("You are not logged in");
    } else {
      account = result[0];
      contract.options.from = account;
    }
  }).catch((error) => {
    console.log("Error: " + error);
  });
  contract.methods.message().call((error, result) => {
      if(error) {
          return console.log(error);
      }
      $('#message').text(result);
  }).catch((error) => {
    console.log("Error: " + error);
  });

  $('#set_message').click(setMessage);
});

function setMessage() {
  let message = $('#new_message').val();
  contract.methods.setMessage(message).send(
    {gasPrice: my_web3.utils.toWei("4.1", 'Gwei')}, 
    (error, result) => {
        if(error) {
            return console.log(error);
        }
        console.log("txhash: " + result); 
    }
  ).catch((error) => {
    console.log("Error: " + error);
  });
}
```
