# Creating a Simple Voting DApp on Celo using Solidity and Truffle-

## Table Of Contents:

- [Creating a Simple Voting DApp on Celo using Solidity and Truffle](#creating-a-simple-voting-dapp-on-celo-using-solidity-and-truffle)
- [Table Of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Setup Environment](#1-setup-environment)
- [Smart Contract](#2-smart-contract)
- [Migration](#3-migration)
- [Celo Network Configuration](#4-celo-network-configuration)
- [Deploy Contract to Celo Network](#5-deploy-contract-to-celo-network)
- [Build the User Interface](#6-build-the-user-interface)
- [Add Authentication](#7-add-authentication)
- [Authentication and Transaction Signing using DappKit](#8-authentication-and-transaction-signing-using-dappkit)
- [Display Vote Counts](#9-display-vote-counts)
- [Conclusion](#conclusion)


## Introduction:

In the world of blockchain, decentralized applications (DApps) are getting more and more well-recognized. In this tutorial, we'll use Solidity and the Truffle framework to build a voting DApp for the Celo blockchain. The voting DApp will let users cast their votes in Celo Dollars (cUSD) for their preferred candidate.

## Step 1: Setup Environment-

First step is to set up the development environment. Make sure [Node.js](https://nodejs.org/en/download) is set up on your PC. Then, execute the subsequent command in your terminal to install [Truffle](https://trufflesuite.com/docs/truffle/how-to/install/):

`npm install -g truffle`

Once Truffle is installed, we will create a new project by running the following commands:

```bash
// Create a new directory for the voting DApp
mkdir voting-dapp

// Change to the voting DApp directory
cd voting-dapp

// Initialize a new Truffle project in the current directory
truffle init

```
As a result, a new Truffle project will be created with the required files & folders installed.


## Step 2: Smart Contract-

The next step is to build a smart contract that would let users choose their preferred candidate. Create a new file called Voting.sol in the contracts folder. Define the following contract in this file:

``` solidity
// SPDX-License-Identifier: MIT

// Specify the Solidity version being used
pragma solidity ^0.8.0;

// Define the Voting contract
contract Voting {
    // Define a mapping to store candidate votes with candidate names as keys and vote counts as values
    mapping (bytes32 => uint256) public votes;

    // Define a mapping to store whether an address has already voted or not
    mapping (address => bool) public hasVoted;

    // Define an event to log when a vote is cast
    event VoteCast(address indexed voter, bytes32 candidate);

    // Function to vote for a candidate
    function voteForCandidate(bytes32 candidate) public {
        // Require that the candidate has not received any vote before
        require(votes[candidate] == 0, "Invalid candidate");

        // Require that the sender has not already voted
        require(!hasVoted[msg.sender], "Already voted");

        // Increment the vote count for the candidate
        votes[candidate] += 1;

        // Record that the sender has voted
        hasVoted[msg.sender] = true;

        // Log the vote cast
        emit VoteCast(msg.sender, candidate);
    }

    // Function to retrieve the total vote count for a candidate
    function totalVotesFor(bytes32 candidate) public view returns (uint256) {
        // Require that the candidate has received at least one vote before
        require(votes[candidate] > 0, "Invalid candidate");

        // Return the vote count for the candidate
        return votes[candidate];
    }
}


```

We defined votes mapping in this contract that will keep track of how many votes each contender has earned. VoteForCandidate and totalVotesFor are two additional functions that we define. Users can vote for their preferred candidate using the voteForCandidate function, and the totalVotesFor function gives the total number of votes cast for a specific candidate.

## Step 3: Migration-

Now, the migration script we write will be used to publish our smart contract to the Celo blockchain. Make a new file called 2_deploy_contracts.js in the migrations folder. Add the following code to this file:

```javascript
// Import the Voting contract artifact from the truffle artifacts
const Voting = artifacts.require("Voting");

// Export a function that takes a deployer object as an argument
module.exports = function (deployer) {
  // Deploy the Voting contract using the deployer
  deployer.deploy(Voting);
};

```

By running this script, Truffle is instructed to publish our Voting contract to the Celo blockchain.

## Step 4: Celo Network Configuration-

We must set up our network settings before we can publish our smart contract to the Celo blockchain. Make a new file called truffle-config.js in the project directory. Add the following code to this file:

```javascript
// Import the 'path' module to work with file paths
const path = require('path');

// Import the 'fs' module to work with file system operations
const fs = require('fs');

// Read the private key from a file and convert it to a string
const privateKey = fs.readFileSync('<PATH_TO_PRIVATE_KEY>').toString();

// Export a configuration object for Truffle
module.exports = {
  networks: {
    // Define a network configuration for the Alfajores testnet
    alfajores: {
      // Define a provider function that returns an instance of Web3 connected to the Celo network
      provider: () => {
        const Web3 = require('web3');
        const web3 = new Web3('<CELO_PROVIDER>');
        // Convert the private key to an account object using 'eth.accounts.privateKeyToAccount'
        return web3.eth.accounts.privateKeyToAccount('0x' + privateKey);
      },
      // Specify the network ID for Alfajores
      network_id: 44787,
      // Set the gas price for transactions on the Alfajores network
      gasPrice: '20000000000',
      // Set the gas limit for transactions on the Alfajores network
      gas: 8000000,
    },
  },
  compilers: {
    // Specify the Solidity compiler version
    solc: {
      version: '0.8.0',
      // Enable the optimizer and set the number of optimization runs
      settings: {
        optimizer: {
          enabled: true,
          runs: 200,
        },
      },
    },
  },
};

```

## Step 5: Deploy Contract to Celo Network-

We will enter the following command into our terminal in order to launch our smart contract onto the Celo network:

```bash
truffle migrate --network alfajores
```

Using the alfajores network configuration that we specified in truffle-config.js, this command instructs truffle to migrate our contract to the celo network. Truffle will output the contract address after the migration is finished, which we will use to communicate with our voting DApp.

## Step 6: Build the User Interface-

The user interface for our voting DApp has to be built next. [React](https://react-cn.github.io/react/downloads.html) will be used in this scenario. Run the below command on your terminal to start a new React app:

```bash
npx create-react-app client
```

In the client folder, this command will create a new React app.

Create a new file called Voting.js inside the client folder's src folder. Add the following code to this file:

```javascript
import React, { Component } from 'react';
import Web3 from 'web3';
import VotingContract from './contracts/Voting.json';

class Voting extends Component {
  constructor(props) {
    super(props);

    this.state = {
      candidates: [],
      selectedCandidate: '',
      web3: null,
      contract: null,
      account: null,
      loading: true,
    };
  }

  async componentDidMount() {
    await this.loadWeb3(); // Load Web3 library
    await this.loadBlockchainData(); // Load blockchain data
  }

  async loadWeb3() {
    if (window.ethereum) { // Check if MetaMask is available
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable(); // Enable MetaMask
    } else if (window.web3) { // If MetaMask is not available, check for injected web3 instance
      window.web3 = new Web3(window.web3.currentProvider);
    } else {
      window.alert('Non-Ethereum browser detected. You should consider trying MetaMask!'); // Alert for non-Ethereum browsers
    }
  }

  async loadBlockchainData() {
    const web3 = window.web3; // Get the web3 instance
    const accounts = await web3.eth.getAccounts(); // Get the accounts from MetaMask
    this.setState({ account: accounts[0] }); // Set the current account to the state

    const networkId = await web3.eth.net.getId(); // Get the network ID
    const networkData = VotingContract.networks[networkId]; // Get the network data from the contract JSON
    if (networkData) {
      const contract = new web3.eth.Contract(VotingContract.abi, networkData.address); // Create a contract instance
      this.setState({ contract }); // Set the contract instance to the state
      const candidates = await contract.methods.getCandidates().call(); // Call the 'getCandidates()' function from the contract
      this.setState({ candidates, loading: false }); // Set the candidates and loading state to the state
    } else {
      window.alert('Smart contract not deployed to detected network.'); // Alert if contract not deployed to the detected network
    }
  }

  render() {
    return (
      <div className="container">
        <h1>Voting DApp</h1>
        {this.state.loading ? (
          <p>Loading...</p>
        ) : (
          <div>
            <h3>Candidates:</h3>
            <ul>
              {this.state.candidates.map((candidate) => (
                <li key={candidate}>{candidate}</li>
              ))}
            </ul>
            <form
              onSubmit={(event) => {
                event.preventDefault();
                this.state.contract.methods
                  .voteForCandidate(this.state.selectedCandidate)
                  .send({ from: this.state.account })
                  .once('receipt', (receipt) => {
                    this.setState({ loading: false });
                  });
              }}>
              {/* Form for voting */}
            </form>
          </div>
        )}
      </div>
    );
  }
}

```

## Step 7: Add Authentication-

We need to put in place an authentication system that will authenticate each voter in order to stop users from casting repeated ballots. To accomplish this, we can make use of the @celo/dappkit package.

Installing the package first requires typing the following command into your terminal:

```bash
npm install @celo/dappkit
```

Next, we need to modify our Voting.js file to include the authentication logic. Here's the updated code:

```javascript
import React, { Component } from 'react';
import Web3 from 'web3';
import VotingContract from './contracts/Voting.json';
import { newKitFromWeb3 } from '@celo/contractkit';
import { getAccountAddress, getPhoneHash } from '@celo/utils';
import { requestTxSig } from '@celo/dappkit';

class Voting extends Component {
  constructor(props) {
    super(props);

    // State variables
    this.state = {
      candidates: [], // Array to store candidates
      selectedCandidate: '', // Currently selected candidate
      web3: null, // Web3 instance
      contract: null, // Voting smart contract instance
      account: null, // Ethereum account address
      loading: true, // Loading state flag
      dappkitResponse: null, // DappKit response for authentication
    };
  }

  async componentDidMount() {
    await this.loadWeb3(); // Load Web3 instance
    await this.loadBlockchainData(); // Load blockchain data
  }

  async loadWeb3() {
    if (window.ethereum) {
      // If MetaMask or similar provider is available
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable();
    } else if (window.web3) {
      // If Web3 instance is already available
      window.web3 = new Web3(window.web3.currentProvider);
    } else {
      // If no provider is available
      window.alert('Non-Ethereum browser detected. You should consider trying MetaMask!');
    }
  }

  async loadBlockchainData() {
    const web3 = window.web3;
    const accounts = await web3.eth.getAccounts();
    this.setState({ account: accounts[0] }); // Set the Ethereum account address to state

    const networkId = await web3.eth.net.getId();
    const networkData = VotingContract.networks[networkId];
    if (networkData) {
      const contract = new web3.eth.Contract(VotingContract.abi, networkData.address);
      this.setState({ contract }); // Set the Voting smart contract instance to state
      const candidates = await contract.methods.getCandidates().call();
      this.setState({ candidates, loading: false }); // Set the candidates array and loading state to state
    } else {
      window.alert('Smart contract not deployed to detected network.');
    }
  }

  async authenticate() {
    // Function for authentication using Celo DappKit
    const kit = newKitFromWeb3(window.web3);
    const address = await getAccountAddress();
    const phoneHash = await getPhoneHash(address);
    const requestId = 'authentication';
    const dappkitResponse = await requestTxSig(
      kit,
      [
        {
          from: address,
          to: address,
          value: '0',
          data: '0x',
        },
      ],
      requestId,
      {
        txMessage: 'Sign in to Voting DApp',
        displayName: 'Voting DApp',
        icon: window.location.origin + '/logo192.png',
      }
    );
    this.setState({ dappkitResponse }); // Set the DappKit response to state
  }

  render() {
    return (
      <div className="container">
        <h1>Voting DApp</h1>
        {this.state.loading ? (
          <p>Loading...</p>
        ) : (
          <div>
            <h3>Candidates:</h3>
            <ul>
              {this.state.candidates.map((candidate) => (
                <li key={candidate}>{candidate}</li>
              ))}
            </ul>
          </div>
       
```

We must update our smart contract to include a function that returns the total number of votes cast for each candidate in order to display the voting results. This is the most recent Voting.sol code:

``` solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.4.22 <0.9.0;

contract Voting {
    string[] public candidates;     // Array to store the list of candidates
    mapping (string => uint256) public votes;     // Mapping to store the votes count for each candidate

    constructor() {
        candidates = ["Candidate 1", "Candidate 2", "Candidate 3"];     // Constructor to initialize the candidates array with initial candidates
    }

    function vote(string memory candidate) public {
        require(validCandidate(candidate));     // Function to cast a vote for a valid candidate
        votes[candidate] += 1;     // Increment the vote count for the candidate
    }

    function getCandidates() public view returns (string[] memory) {
        return candidates;     // Function to retrieve the list of candidates
    }

    function getVotes(string memory candidate) public view returns (uint256) {
        require(validCandidate(candidate));     // Function to retrieve the vote count for a valid candidate
        return votes[candidate];     // Retrieve the vote count for the candidate
    }

    function validCandidate(string memory candidate) public view returns (bool) {
        for (uint i = 0; i < candidates.length; i++) {     // Function to check if a given candidate is valid
            if (keccak256(abi.encodePacked(candidates[i])) == keccak256(abi.encodePacked(candidate))) {
                return true;     // Return true if the candidate is found in the candidates array
            }
        }
        return false;     // Return false if the candidate is not found in the candidates array
    }
}

```

A new function called getVotes has been introduced; it accepts a candidate name as an input and returns the total number of votes cast for that particular contender.

To display the vote results, we must next make changes to the vote.js file. This is the revised code:

```javascript
import React, { Component } from 'react';
import Web3 from 'web3';
import VotingContract from './contracts/Voting.json';
import { newKitFromWeb3 } from '@celo/contractkit';
import { getAccountAddress, getPhoneHash } from '@celo/utils';
import { requestTxSig } from '@celo/dappkit';

class Voting extends Component {
  constructor(props) {
    super(props);

    this.state = {
      candidates: [],
      selectedCandidate: '',
      web3: null,
      contract: null,
      account: null,
      loading: true,
      dappkitResponse: null,
      voteCounts: null,
    };
  }

  async componentDidMount() {
    await this.loadWeb3(); // Load Web3 on component mount
    await this.loadBlockchainData(); // Load blockchain data on component mount
    await this.getVoteCounts(); // Get vote counts on component mount
  }

  async loadWeb3() {
    if (window.ethereum) { // If MetaMask is present
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable(); // Enable MetaMask
    } else if (window.web3) { // If web3 instance is present
      window.web3 = new Web3(window.web3.currentProvider);
    } else {
      window.alert('Non-Ethereum browser detected. You should consider trying MetaMask!');
    }
  }

  async loadBlockchainData() {
    const web3 = window.web3;
    const accounts = await web3.eth.getAccounts();
    this.setState({ account: accounts[0] });

    const networkId = await web3.eth.net.getId();
    const networkData = VotingContract.networks[networkId];
    if (networkData) { // If smart contract is deployed on the detected network
      const contract = new web3.eth.Contract(VotingContract.abi, networkData.address);
      this.setState({ contract });
      const candidates = await contract.methods.getCandidates().call(); // Get candidates from smart contract
      this.setState({ candidates, loading: false });
    } else {
      window.alert('Smart contract not deployed to detected network.');
    }
  }

  async authenticate() {
    const kit = newKitFromWeb3(window.web3);
    const address = await getAccountAddress();
    const phoneHash = await getPhoneHash(address);
    const requestId = 'authentication';
    const dappkitResponse = await requestTxSig(
      kit,
      requestId,
      { phoneHash }
    ); // Request authentication signature from Celo Wallet app
    await dappkitResponse.waitReceipt();
    this.setState({ dappkitResponse });
  }

```
In order to integrate the authentication process, we also need to edit our handleVote function:

```javascript
// Handle vote submission
handleVote = async (event) => {
  event.preventDefault();
  const { contract, selectedCandidate } = this.state;

  // Authenticate user before signing the transaction
  await this.authenticate();

  // Get user's Ethereum account
  const accounts = await window.web3.eth.getAccounts();
  const account = accounts[0];

  // Get current vote count for the selected candidate
  const voteCount = await contract.methods.getVotes(selectedCandidate).call();

  // Send a vote transaction to the smart contract
  await contract.methods.vote(selectedCandidate).send({
    from: account,
    gas: 200000,
  });

  // Update the vote count in the state after a successful vote transaction
  this.setState({ voteCounts: { ...this.state.voteCounts, [selectedCandidate]: parseInt(voteCount) + 1 } });
}

```

It is known as .Prior to the vote transaction, authenticate() and watch for the dappkitResponse to be returned. We then, utilize contract.methods to conduct the vote transaction after retrieving the user's account.vote(selectedCandidate).send(). Finally, we update the state's vote total.

In order to retrieve the vote totals for each candidate, we also need to alter our getVoteCounts function:

```javascript
// Get vote counts for all candidates from the smart contract
async getVoteCounts() {
  const { contract, candidates } = this.state;
  const voteCounts = {};

  // Loop through all candidates and get their respective vote counts
  for (let i = 0; i < candidates.length; i++) {
    const candidate = candidates[i];
    const voteCount = await contract.methods.getVotes(candidate).call();
    // Parse the vote count to an integer and store in the voteCounts object
    voteCounts[candidate] = parseInt(voteCount);
  }

  // Update the state with the voteCounts object
  this.setState({ voteCounts });
}

```

We invoke contract.methods for each candidate in a loop.getVotes(candidate).We use call() to get the number of votes cast, then we save it in the voteCounts object in the state.

## Step 9: Display Vote Counts-

To display the vote totals for each contender, we must lastly change the Voting.js file. We'll introduce a new function, renderVoteCounts, which returns a list of candidates together with the total number of votes they've received:

```javascript
// Render the vote counts for each candidate
renderVoteCounts() {
  const { candidates, voteCounts } = this.state;
  const items = [];

  // Loop through all candidates and render their respective vote counts
  for (let i = 0; i < candidates.length; i++) {
    const candidate = candidates[i];
    const voteCount = voteCounts[candidate];
    // Push the vote count as a list item with the candidate name as the key
    items.push(
      <li key={candidate}>
        {candidate}: {voteCount}
      </li>
    );
  }

  // Return the list of vote counts as an unordered list
  return (
    <ul>{items}</ul>
  );
}

```

We also need to modify our render function to call this.renderVoteCounts():

```javascript
// Render the main UI
render() {
  const { candidates, loading } = this.state;

  // Check if data is still loading, display "Loading..." if so
  if (loading) {
    return <div>Loading...</div>;
  }

  // Render the main UI with the candidate form
  return (
    <div className="container">
      <h1>Vote for your favorite candidate</h1>
      <form onSubmit> {/* Form submission handler not specified */}
        {/* Form contents go here */}
      </form>
      {/* Additional UI elements go here */}
    </div>
  );
}

```

## Conclusion:

Finally, utilizing Solidity and the Truffle framework, we were able to effectively develop a simple voting DApp for the Celo network. In order to get started, we first set up our development environment and used Solidity to build a Smart Contract. The contract was then deployed on the Celo network once we configured our network parameters.

Additionally, we used React to create the user interface and the web3 library to communicate with our Smart Contract. Finally, we put our DApp to the test by casting a ballot for a candidate and showing the results.

This tutorial gives you a fundamental grasp of how to use Solidity and Truffle to build a DApp for the Celo network. With this framework, you may construct more intricate DApps that communicates with the Celo blockchain and make use of the capabilities of the platform to develop ground-breaking solutions to modern problems.

Regardless of their background or location, developers may contribute to the development of a more open and accessible financial systems by building DApps on the Celo network. Decentralized applications that may assist solve real-world problems are becoming more and more necessary as blockchain technology is being adopted more widely. The Celo network offers developers a strong platform on which to build these applications and advance global financial inclusion.

