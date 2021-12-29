# Blockchain example with reactjs , ganache , truffle , smartcontract example for beginners

# ganache  :
1. install and start ganache on your local network 
2. from its settings frin the port and local hostedd details 

# start truffle 
1. Create a project folder with project name 
 2. inside the project folder first initialize truffle

        truffle init

 3. Then initialize react in the same directory

        npx create-react-app client  

 4. install react web3
 
        npm install web3

 5. write program in react js . 
 
 6. Now start ganache 
 * Go to the truffle directory 
 * Open truffle-config.js and uncomment these lines and change port 
  ```js

     contracts_build_directory: "./src/abi",

     development: {
      host: "127.0.0.1",     // Localhost (default: none)
      port: 7545,            // Standard Ethereum port (default: none)
      network_id: "*",       // Any network (default: none)
     },

  ```


 * type the command : 

        truffle compile

        truffle migtate --reset
        
This will compile and deploy our smart contract 
* After compiling there will be a folder called 'build' in the truffle directory 
* Open that , There will be a file called 'Migration.js' . 

# To Create our own smart contract :
  To create and deploy our own smart contract we need to create a .sol file in the folder contractes . 
  Here we create a file Test_migrations.sol and we paste the below code 

  ```js
    pragma solidity >=0.4.22 <0.9.0;

    contract Test_contract 
    {
      address public owner = msg.sender;

      function testFunction () public pure returns(string memory ){
          return "hello from first test";
      }
    }
  ```

  To deploy the contract create a migraion file in the migrations folder
  There will be one present there named '1_initial_migration'
  It is already defined 

  To deploy our contact we need to create one file with a name that you want 
  Here we create : '2_test_migration.js'
  inside that file we write codes to migration

  ```js
    const Test_contract = artifacts.require("Test_contract");
    // Test_contract is the name of the contract saved inside contracts folder
    module.exports = function (deployer) {
      deployer.deploy(Test_contract);
    };
  ```

  then type the commands 

        truffle compile

        truffle migtate --reset

  If it migrate successfully there will be a file called  `Test_contract.json`  inside `'src/abi'` folder




# Connect metamask from chrome to ganache local network

* Click on custom rcp 
* enter the host address name and etc..
* chain id will be 1337 default for localhost

* now click on the round symbol in metamask
* click on import account 
* Open ganache , tap on the key symbol of the address that you wanna use , It will show a private key , copy and paste that private key to metamask 


# Connecting metamask with React :
8. Write react functions to connect to the metamask account

import web3 first .

The below code is the base functions to get balance , connect to wallet and etc..

```js
const web3 = new Web3(Web3.givenProvider || "http://localhost:7545");

async function LoadBlockChain()
  {
    
    // connects to the local evelopment environment



    const ConnectToMetaMask = async () => {
      
      // This function is used to check if metamask is connected or not 

      if (window.ethereum) 
      {
        // the function enters here if metamask is installed
        console.log("metamask is present ")
        await window.ethereum.send('eth_requestAccounts');
        window.web3 = new Web3(window.ethereum);

        // console.log(window.web3);
        const acc = await web3.eth.getAccounts();

        console.log(acc)
        // The address is then stored 
        setAddress(acc)

        // In this case, the await window.ethereum.send('eth_requestAccounts') function calls the pop-up UI dialogue that asks the user’s permission to connect the dApp to MetaMask.

        setIsConnectedToWallet(true);
        // if we connect to the wallet successfully we set isConnectedToWallet to true


        // This function returns true if metamask is installed
        return true;
      }
      else{
        // the function enters here if metamask is not installed
        console.log("metamask is not installed")
      }

      // This function returns false if metamask is not installed

      return false;
    }

   await  ConnectToMetaMask()

  }






  const getBalance = async () => {
    
    if(!isConnectedToWallet)
    {
      // If wallet is not connected we enter here 
      console.log("Please connect to wallet ");
      return;
    }

    console.log("address : " , address);
    

    // prints the balance of the connected address
    const balanceFrom = web3.utils.fromWei(
       await web3.eth.getBalance( address[0] ),
       'ether'
    );

    console.log("balance is :" ,  balanceFrom )
    
 };


```



# Connect to our smart contract . 
To connect to our own smart contract we need to import the `'src/abi/Test_contract.json';` to our code which contains the informations to the connection , such as block addres ..

So first we import the file to our `App.js`
`import Test_contract from '../abi/Test_contract.json';` 


To connect to the smart contract we need to use the below syntax 
```js 
  new web3.eth.Contract(abi , address)
```

here abi is `Test_contract.abi` and address can be found from the `'src/abi/Test_contract.json';` file . Go to that file and search for address , copy that address and paste it in the address field .

We wrtite a function to do this all and it looks like below 

```js
  const ConnectSmartContract = async () =>{
    // This function is used to connect to the smart contract 
    if(!isConnectedToWallet)
    {
      // If wallet is not connected we enter here 
      console.log("Please connect to wallet ");
      return;
    }


    // These below lines are used to fetch the address of our smart contract
    // The address is present inside the test_contract.json 

    const networkId = await web3.eth.net.getId()
    console.log( "networkId :  " , networkId );
    const networkData = Test_contract.networks[networkId];
    console.log( "network Data :  " , networkData);

    if(networkData)
    {
      var smartContractAddress = networkData.address;
      console.log( "network address :  " , smartContractAddress);
      
      // the below line is used to connect to the Test_contract , 
      // ie , our smart contract
      const TestContract =  new web3.eth.Contract( Test_contract.abi , smartContractAddress );

      console.log(TestContract);

      return TestContract;
    }
    else{
      console.log("not found");
    }

  

  }

  const smartContractFunctionCall = async () =>{
    // This function is used to call the functions in the smart contract 
    const TestContract = await ConnectSmartContract();
    // above line connect to the smart contract
    //  ConnectSmartContract() is a function we have written to connect with our smart contract 


    const dataBack = await TestContract.methods.testFunction().call();

    // This runs the TestContract and stores the return value to dataBack variable 
    

    console.log("dataBack : " , dataBack );


  }
```


On the console we can see 
```
 dataBack :  hello from first test
```

Which is the data returned from the smart contract 
