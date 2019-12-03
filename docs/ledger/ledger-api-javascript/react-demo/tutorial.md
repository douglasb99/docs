# Overview

The [Ledger API Javascript React Node.js](https://github.com/fetchai/javascript-api-demo) is a demo of 
the [Ledger API Javascript](https://github.com/fetchai/ledger-api-javascript).
The Ledger API Javascript is a browser and Node.js capable NPM module [fetchai-ledger-api](https://www.npmjs.com/package/fetchai-ledger-api) 
which provides high level methods to allow simple interaction with the [Fetch Ledger](https://docs.fetch.ai/). 

Functionality of the [Ledger API Javascript](https://github.com/fetchai/ledger-api-javascript) includes:

- Generation of new Addresses and their associated private keys.
- Checking balances (Fet in 10^-10) held at accounts, using Addresses.
- Transferring Fet between accounts.
- Submitting smart contracts to the Fetch Ledger.
- Finding available servers to connect to on the fetch network(termed Bootstrapping).

Functionality covered in this [demo](https://github.com/fetchai/javascript-api-demo) includes:

- Generation of new Addresses and their associated private keys.
- Checking balances (Fet in 10^-10) held at accounts.
- Finding available servers to connect to (termed Bootstrapping).    

## Installation of Node
Use node version manager (nvm) to install latest stable version of npm:
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
nvm install node
nvm use node
nvm install --lts
nvm use --lts
```

## Installation of Project
```
git clone https://github.com/fetchai/javascript-api-demo
npm install 
```
## RunNing the Demo

```
npm run build
npm run start
```

## Requiring the fetchai-ledger-api NPM Module 

- Whilst in this demo  `npm install` handles the installation of our NPM Module use the below command to install it in your own project
```
npm install -i fetchai-ledger-api
``` 

- One can  require the address class like so
```import {Address} from "fetchai-ledger-api";```

- In this demo we import four classes as below
```
import {Bootstrap, Entity, Address, LedgerApi} from "fetchai-ledger-api";
```
   
## Working with Addresses

### Overview

- An account consist of an Address and a private key. An Address is a base-58 encoded string containing a public key and allows one to transfer Fet into an account and one to look-up the balance held by an account.
- A private key allows for withdrawal of funds, and must be kept secret. 
- The Entity class represents a public/private key pair and provides for some basic operations pertaining to this such as key generation and the generation of a 
  an Address from a private key. 
  
- The Address class wraps the public address, which it takes as a constructor, and is useful for validating and converting between Address formats.
 
### Code Explanation
 
- Create a new Private key by instantiating an Entity object without passing any parameters:

```
const entity = new Entity()
```
- Return the the private key in hexadecimal    
```
const private_key_hex = entity.private_key_hex()
```
- We can also return the key as a buffer
```
const private_key_buffer = entity.private_key()
  ``` 
- Which can be converted to a base 64 string
```
const base64_data = private_key_buffer.toString('base64')
  ``` 
- We can construct an Entity from a buffer holding a private key
```
const entity = new Entity(private_key)
```
- We can also construct it from a base64 string
```
const other = Entity.from_base64(base64_data)
``` 
- To get the Address of a our user pass the entity as a parameter to the new Address constructor like so
```
const address = new Address(entity).toString()
```

- One can validate the address is of the correct format by wrapping the "new Address" invocation
 in a try/catch block, which will throw a ValidationError if the given address is invalid. Alternatively one can 
 use the below function
 
```
valid_address(display) {
        const bytes = bs58.decode(display)
        return bytes.length === DISPLAY_BYTE_LENGTH
    }
```
    
This requires [bs58](https://www.npmjs.com/package/bs58) which can be installed and configured as below

```
npm i --save bs58
npm install -g browserify
browserify node_modules/bs58/index.js -o bs58.bundle.js --standalone bs58
```

## Connecting to a Server (Bootstrapping)

### Overview

Whilst anybody may run a node in the [Fetch Ledger](https://docs.fetch.ai/) there are some publicly available servers
whose Addresses can be found using our Bootstrapping functionality. 

Bootstrapping works by one passing in the name of a publicly 
available server on the Fetch network and bootstrapping returns this Server's Host and Port. 

The Host and Port of a server on the network is needed to perform 
operations against the Ledger such as querying an account balance, transferring funds or submitting a smart contract to the Ledger.  

- Importing the Bootstrapping Class
```
import {Bootstrap} from "fetchai-ledger-api";
```
- One can pass in the name of a server ran by Fetch.ai such as 'devnet'.
```        
const [host, port] = await Bootstrap.server_from_name("devnet")
```
- In our React Demo we use the async/await syntax to handle the promise returned however one can use the then 
  syntax.
```  
const promise = fetchai.Bootstrap.server_from_name('betanet')
promise.then(function (result) {
    console.log("Betanet is available as host" + result[0] + " and port " + result[1])
})
```  

## Querying the Balance of an Account

### Overview

Balances are stored as integers in 10^-10 Fet, and can be queried by anybody with an Address.  

### Code Explanation

- Import the LedgerApi class
```
import {LedgerApi} from "fetchai-ledger-api";
```
- Create a new Ledger Object with the host and port of an available server. 
- One can also run the [Fetch Ledger](https://docs.fetch.ai/) locally during development and use the value "localhost" along with the port it is running on. 
```
 const api = new LedgerApi(host, port)
```
- To Query the balance (returning an integer value equal to the quantity of Nano Fet held at the account) call the balance method with the public Address 
``` 
      let balance = null;
        try {
            balance = await this.api.tokens.balance(this.state.public_key_display)
        } catch (e) {
            // error handling logic in-case key is invalid, or network request fails.
            this.setState({error_message: e.message});
        }
```

