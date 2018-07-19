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


### 1.  Run `npm init`
This command will create a `package.json` file, required for managing the packages we will be adding below.

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

### 6.  Launch the server with `npx webpack-dev-server --https`

Same as last time.

<br>

### 7.  Build with `npx webpack --mode=development`

Execute this command in a new command window so that you don't need to stop the server.

This compiles your application, generating the `dist/main.js` required when viewing the site.

This command needs to be repeated anytime you change a js file.  There are hot-reloading options available to make development easier (included with the Vuejs template mentioned above). 

### 8.  Test

At this point, the app should work exactly as it did at the end of Part 2.  It should work so long as the user has Metamask installed and the correct network selected.

<h2>Add Ledger Support</h2>

### 9. Install web3 and the Ledger components

```
npm install web3 web3-provider-engine @ledgerhq/web3-subprovider  @ledgerhq/hw-transport-u2f
```

#### `web3`

We need to install Web3.js to use if Metamask is not available. 

#### `web3-provider-engine`

This allows us to use Web3 with custom providers.  We use this to enable two features: 1) Ledger integration and 2) read-only support when neither Metamask nor Ledger is available.

#### `@ledgerhq/web3-subprovider  @ledgerhq/hw-transport-u2f`

These two components are required to interface with the Ledger Nano S hardware wallet.

<br>

### 10.  Import Web3.js

Add the following to the top of `index.js`:
 
```javascript
import Web3 from "web3"
```

<br>

### 11.  Declare global variables

let my_web3;
let account;
const rpcUrl = "https://ropsten.infura.io";

#### `my_web3`

Metamask does not use the latest version of Web3.js, presumably to maintain backwards compatibility for dapps.  In order to have consistent code when working with either Metamask or Ledger, we will be replacing the `web3` object Metamask provides with this `my_web3` object.

#### `account`

By pulling in the Web3.js library, we are able to support read-only calls for users which don't have either Metamask or Ledger connected (i.e. anyone can read the current message).  

This variable allows us to block any attempt to create a transaction (which would fail anyways).

#### `rpcUrl`

Our application is reading information from an Ethereum node.  Metamask will select the node automatically, but we need to do it ourselves for users without Metamask.


<br>

### 12.  Construct the `my_web3` object

Remove the 'Metamask is not installed' check, it's no longer an error as we can support users without Metamask.  Replace it with the following:

```javascript
window.addEventListener('load', () => {
  if(typeof(web3) == 'undefined') {
    my_web3 = new Web3(new Web3.providers.HttpProvider(rpcUrl));
  } else {
    my_web3 = new Web3(web3.currentProvider);    
  }

  ...
}
```

<br>

### 13.  Update the Web3.js calls to the latest standard

```
  contract = new my_web3.eth.Contract(abi, contract_address);
  contract.methods.message().call((error, result) => {
    ...
  });

  ...

  contract.methods.setMessage(message).send(
    {gasPrice: my_web3.utils.toWei("4.1", 'Gwei')},
    ...
  );
```






<hr>

<br>

That’s it!  Hope this was helpful.  

<br>

<hr>

<h1>Source Code - Part 3</h1>

```html

```
