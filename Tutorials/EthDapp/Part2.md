*Note: We **highly** recommend viewing this tutorial in Light Mode. Steemit does not support dark themes for inline and code blocks, making this page difficult to read in Night Mode.*<br>

<hr>

In this tutorial series, we will be creating a dapp for Ethereum.  It will be broken into three parts: 
1.  [The Smart Contract](https://steemit.com/tutorial/@hardlydifficult/ethereum-dapp-tutorial-part-1-of-3-solidity-smart-contract)
2.  **Web front-end with Metamask integration** (this part)
3.  Ledger integration

We’ll be creating a simple dapp called ‘Message of the Moment,’ which will display a message that anyone is welcome to change.

Full source code for Part 2 is at the bottom of the post.

<hr>

<h1>Part 2: Web front-end with Metamask integration</h1>
This tutorial will utilize the following references and resources. Don’t worry about collecting them all right now; we’ll refer to them inline as we use them.

* [NPM and NodeJS](https://www.npmjs.com)
* [Webpack](https://webpack.js.org/guides/getting-started/)
* [Web3.js](https://github.com/ethereum/web3.js/)
* [jQuery](https://jquery.com/)
<br>

<h2> Setting Up the Environment</h2>

### 1.  Install [NPM and Node.js](https://www.npmjs.com/get-npm)
Metamask will only communicate with websites using https.  We’ll be using Node.js as our webserver so we can test https locally.

<br>
### 2.  Create a folder for your dapp, and a directory inside named 'src'.
![directory structure](https://i.imgur.com/2tSSdWG.png)

<br>
### 3.  Create an empty file in the src directory named 'index.js'.
This file, `src/index.js` is required in order to start the Webpack dev server.  We only need the most basic support from Webpack, so this file is going to remain empty for this tutorial.

<br>
### 4.  Install Webpack.
Open a cmd prompt, switch to dapp folder and run the following:
```
npm install webpack webpack-cli webpack-dev-server --save-dev
```

#### `npm`
NPM is a package manager for Javascript.  It's purpose is to make retrieving dependencies for your project, such as Webpack, easy.

#### `install`
`install` in NPM means it will download the component and save it in the current directory.  So this is something you would repeat per-project.

#### `webpack...`
We are using Webpack as the local server for development and testing.  We'll need all 3 of these Webpack components for this.

#### `--save-dev`
`--save-dev` tells NPM to make the packages available in development mode, but not while in production mode.  The dapp we are creating will have no server-side dependencies.

<br>
### 5.  Launch the Webpack dev server.
```
npx webpack-dev-server --https
```

#### `npx`
`npx` is the command used to execute a NPM package, in this case the `webpack-dev-server`.

#### `--https`
The --https option specified here will generate a certificate automatically, making testing easy.

<br>
### 6.  Open https://localhost:8080 in your browser.
You can click past the scary warnings by selecting 'Advanced' followed by 'Proceed to localhost (unsafe)'. This warning message will only appear when testing locally.  Once the app is deployed, users will not see this.

<h2>Create the dapp’s webpage</h2>

### 7. Create index.html in the dapp's directory.
Add the following template of a basic webpage with jQuery to build from:
```html
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
  <script>
  <\script>
<\head>
<body>
<\body>
<\html>
```

<br>
### 8.  Add the contract’s address from Remix.

We will need the contract's address below, save it as a const named `contract_address`.  Note that if your dapp supports both testnet and mainnet, you will need a way of selecting the correct address for the selected network.

```html
  <script>
    const contract_address = "0x654b54c945d29981d597fc8756cdb3c6e372440c";
  </script>
```

<br>
### 9.  Copy the ABI from Remix.
To get the `abi`, click the ‘Compile’ tab, select ‘Details’ and then click the copy icon next to ‘ABI’.  Paste this as a `const` named `abi` under the `contract_address`.  

The ABI may be quite long.  You may want to save this in a separate file for readability. 

```javascript
const abi = [
	{
		"anonymous": false,
		"inputs": [
	...
	}
];
```

#### ABI
ABI, or Application Binary Interface, describes the API supported by your contract.  We use this call methods or read data types by name.

<br>
### 10.  Add a div to display the message.
```html
<body>
  <div id="message"></div>
</body>
```

<br>
### 11.  Create a `contract` object
Metamask automatically adds an object named `web3` to the page before the ‘`load`’ event completes.  On `load`, we leverage the `web3` object to create a `contract` object using the `abi` and `contract_address` defined above.  

If the `web3` object is not found, it is because the user does not have Metamask installed.  We will address this scenario in part 3.
```javascript
window.addEventListener('load', () => {
      if(!web3) {
	    return console.log("Metamask is not installed");
      }
      contract = web3.eth.contract(abi).at(contract_address);
});
```

<br>
### 12.  Read the current message.
After the `contract` object has been created, we can use it to read data, call methods, and post transactions through Metamask.  Here we are reading `message`, which is a `string` in the `contract`.
```javascript
window.addEventListener('load', () => {
    if(!web3) {
        return console.log("Metamask is not installed");
    }
    contract = web3.eth.contract(abi).at(contract_address);

    contract.message.call((error, result) => {
        if(error) {
            return console.log(error);
        }
        $('#message').text(result);
    });
});
```

#### `.call`
`call` is a  [web3.js function](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethcall) which will read a data type or the result from a read-only method.  This will not create a transaction, does not cost gas, and returns very quickly.

<br>
### 13.  Add a form allowing users to change the message.
```html
<br>
New Message: <input type="text" id="new_message" />
<button onclick="setMessage()">Set Message</button>
```

<br>
### 14.  Create the `setMessage` `function` for the form to call.
```javascript
function setMessage() {
    let message = $('#new_message').val();
    contract.setMessage.sendTransaction(
        message, 
        {gasPrice: web3.toWei(4.1, 'Gwei')}, 
        (error, result) => {
            if(error) {
                return console.log(error);
            }
            console.log("txhash: " + result); 
        }
    );
}
```

#### `$('#new_message').val()`
This uses jQuery in order to read the value in the text box.


### `.sendTransaction(params, options, callback)`
`sendTransaction` is a  [web3.js function](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethsendtransaction) which will cause Metamask to prompt the user for confirmation.  We are passing one parameter in, the new message.  Options are optional, here we use it to get a nice default on the gas price of 4.1 Gwei, which must be converted to wei for the API call.


The callback will return the transaction’s hash code, or message any errors.  You can use the transaction’s hash to monitor its status, and automatically refresh the page as appropriate.  This is out of scope for this simple tutorial, but as a next step consider [Web3.js's `getTransactionReceipt` function](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgettransactionreceipt)

<h2>Test and Deploy</h2>

### 15. Using Chrome's Developer Tools
Open 'Developer Tools' in Chrome when debugging.  The 'Console' may display helpful information.  You can also use the 'Sources' tab to add a breakpoint by clicking a line number in your code.  This will pause the site on that line and allow you to mouse over data for more information and to step through the logic.

### 16. Deploy to GitHub
Obviously there are tons of hosting options.  I like GitHub as a free and easy way to host a website, and it has https support which is required for our dapp.

Create a repository on GitHub and upload your index.html file (and any other dependencies such as images you may have added).  You can upload using the 'Upload Files' button if you are not familiar with Git.

Then open 'Settings' and under 'GitHub Pages' 'Source' select 'master branch' and hit 'Save'.  A URL will appear, but it may take a few minutes before the site is available.

<hr>
<br>

That’s it!  Hope this was helpful.  In Part 3 we will be adding support for the Ledger Nano S hardware wallet, and addressing some of the corner cases including getting the message to display to someone without either Metamask or a Ledger.  

<br>
<hr>

<h1>Source Code - Part 2</h1>

```html
<html>
<head>
	<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
  <script>
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
	

    let contract;
    window.addEventListener('load', () => {
			if(!web3) {
				return console.log("Metamask is not installed");
			}
      contract = web3.eth.contract(abi).at(contract_address);
      contract.message.call((error, result) => {
				if(error) {
					return console.log(error);
				}
				$('#message').text(result);
      });
    });

    function setMessage() {
			let message = $('#new_message').val();
			contract.setMessage.sendTransaction(
				message, 
				{gasPrice: web3.toWei(4.1, 'Gwei')}, 
				(error, result) => {
					if(error) {
						return console.log(error);
					}
					console.log("txhash: " + result); 
				}
			);
    }
  </script>
</head>
<body>
  <div id="message"></div>

  <br>
  New Message: <input type="text" id="new_message" />
  <button onclick="setMessage()">Set Message</button>
</body>
</html>
```
