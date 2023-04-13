# What is a Voting DApp?

A Voting DApp is a decentralized application built on a blockchain network that allows users to participate in a secure and transparent voting process without the need for a centralized authority.

In a Voting DApp, the voting process is conducted on a blockchain network, which ensures that the voting results are tamper-proof and cannot be altered by any party. The voters can cast their votes anonymously, and the results are publicly visible for anyone to verify.

Voting DApps can be used for various purposes, such as elections, surveys, decision-making processes, and more. They can be designed to accommodate different voting methods, such as first-past-the-post, ranked-choice, and more.

Some popular examples of Voting DApps include Horizon State, Votem, and Agora Voting.

# Creating a Simple Voting DApp on Celo using Solidity and Truffle

## Table Of Contents

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
- [Authentication and Transaction Signing using DappKit](#9-authentication-and-transaction-signing-using-dappkit)
- [Display Vote Counts](#10-display-vote-counts)
- [Conclusion](#conclusion)


## Introduction

In the world of blockchain, decentralized applications (DApps) are getting more and more well-liked. In this tutorial, we'll use Solidity and the Truffle framework to build a straightforward voting DApp for the Celo blockchain. The voting DApp will let users cast their votes in Celo Dollars (cUSD) for their preferred candidate.

## 1: Setup Environment

Setting up the development environment is the first step. Make sure Node.js is set up on your PC. then execute the subsequent command in your terminal to install Truffle:

`npm install -g truffle`

Once Truffle is installed, we will create a new project by running the following commands:

```bash
mkdir voting-dapp
cd voting-dapp
truffle init
```

With the required files and folders, a new Truffle project will be created as a result.

## 2: Smart Contract

The next step is to build a smart contract that would let users choose their preferred candidate. Create a new file called Voting.sol in the contracts folder. Define the following contract in this file:

```solidity
pragma solidity ^0.8.0;

contract Voting {
    mapping (bytes32 => uint256) public votes;

    function voteForCandidate(bytes32 candidate) public {
        require(votes[candidate] != 0, "Invalid candidate");
        votes[candidate] += 1;
    }

    function totalVotesFor(bytes32 candidate) public view returns (uint256) {
        require(votes[candidate] != 0, "Invalid candidate");
        return votes[candidate];
    }
}
```

We defined a votes mapping in this contract that will keep track of how many votes each contender has earned. VoteForCandidate and totalVotesFor are two additional functions that we define. Users can vote for their preferred candidate using the voteForCandidate function, and the totalVotesFor function gives the total number of votes cast for a specific candidate.

## 3: Migration

The migration script we write next will be used to publish our smart contract to the Celo blockchain. Make a new file called 2_deploy_contracts.js in the migrations folder. Add the following code to this file:

```javascript
const Voting = artifacts.require("Voting");

module.exports = function (deployer) {
  deployer.deploy(Voting);
};
```

By running this script, Truffle is instructed to publish our Voting contract to the Celo blockchain.

## 4: Celo Network Configuration

We must set up our network settings before we can publish our smart contract to the Celo blockchain. Make a new file called truffle-config.js in the project directory. Add the following code to this file:

```javascript
const path = require("path");

const fs = require("fs");

const privateKey = fs.readFileSync("<PATH_TO_PRIVATE_KEY>").toString();

module.exports = {
  networks: {
    alfajores: {
      provider: () => {
        const Web3 = require("web3");
        const web3 = new Web3("<CELO_PROVIDER>");
        return web3.eth.accounts.privateKeyToAccount("0x" + privateKey);
      },
      network_id: 44787,
      gasPrice: "20000000000",
      gas: 8000000,
    },
  },
  compilers: {
    solc: {
      version: "0.8.0",
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

## 5: Deploy Contract to Celo Network

We will enter the following command into our terminal in order to launch our smart contract onto the Celo network:

```bash
truffle migrate --network alfajores
```

Using the alfajores network configuration that we specified in truffle-config.js, this command instructs truffle to migrate our contract to the celo network. Truffle will output the contract address after the migration is finished, which we will use to communicate with our voting DApp.

## 6: Build the User Interface

The user interface for our voting DApp has to be built next. React will be used in this scenario. Run the following command in your terminal to start a new React app:

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
    await this.loadWeb3();
    await this.loadBlockchainData();
  }

  async loadWeb3() {
    if (window.ethereum) {
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable();
    } else if (window.web3) {
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
    if (networkData) {
      const contract = new web3.eth.Contract(VotingContract.abi, networkData.address);
      this.setState({ contract });
      const candidates = await contract.methods.getCandidates().call();
      this.setState({ candidates, loading: false });
    } else {
      window.alert('Smart contract not deployed to detected network.');
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
              }}
```

## 7: Add Authentication

We need to put in place an authentication system that authenticates each voter in order to stop users from casting repeated ballots. To accomplish this, we can make use of the @celo/dappkit package.

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

    this.state = {
      candidates: [],
      selectedCandidate: '',
      web3: null,
      contract: null,
      account: null,
      loading: true,
      dappkitResponse: null,
    };
  }

  async componentDidMount() {
    await this.loadWeb3();
    await this.loadBlockchainData();
  }

  async loadWeb3() {
    if (window.ethereum) {
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable();
    } else if (window.web3) {
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
    if (networkData) {
      const contract = new web3.eth.Contract(VotingContract.abi, networkData.address);
      this.setState({ contract });
      const candidates = await contract.methods.getCandidates().call();
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
    this.setState({ dappkitResponse });
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
```

We must update our smart contract to include a function that returns the total number of votes cast for each candidate in order to display the voting results. This is the most recent Voting.sol code:

```solidity
pragma solidity >=0.4.22 <0.9.0;

contract Voting {
    string[] public candidates;
    mapping (string => uint256) public votes;

    constructor() {
        candidates = ["Candidate 1", "Candidate 2", "Candidate 3"];
    }

    function vote(string memory candidate) public {
        require(validCandidate(candidate));
        votes[candidate] += 1;
    }

    function getCandidates() public view returns (string[] memory) {
        return candidates;
    }

    function getVotes(string memory candidate) public view returns (uint256) {
        require(validCandidate(candidate));
        return votes[candidate];
    }

    function validCandidate(string memory candidate) public view returns (bool) {
        for (uint i = 0; i < candidates.length; i++) {
            if (keccak256(abi.encodePacked(candidates[i])) == keccak256(abi.encodePacked(candidate))) {
                return true;
            }
        }
        return false;
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
    await this.loadWeb3();
    await this.loadBlockchainData();
    await this.getVoteCounts();
  }

  async loadWeb3() {
    if (window.ethereum) {
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable();
    } else if (window.web3) {
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
    if (networkData) {
      const contract = new web3.eth.Contract(VotingContract.abi, networkData.address);
      this.setState({ contract });
      const candidates = await contract.methods.getCandidates().call();
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
      Kit)
```

## 9: Authentication and Transaction Signing using DappKit

We need to first authenticate the user before we can sign transactions. The @celo/dappkit library can be used to ask the Celo Wallet app for a signature.

In our Voting.js file, we'll add a new function called authenticate that will utilize requestTxSig to launch the Celo Wallet app and request a signature:

```javascript
async authenticate() {
    const kit = newKitFromWeb3(window.web3);
    const address = await getAccountAddress();
    const phoneHash = await getPhoneHash(address);
    const requestId = 'authentication';
    const dappkitResponse = await requestTxSig(
      kit,
      requestId,
      { phoneHash }
    );
    await dappkitResponse.waitReceipt();
    this.setState({ dappkitResponse });
  }
```

In order to integrate the authentication process, we also need to edit our handleVote function:

```javascript
handleVote = async (event) => {
  event.preventDefault();
  const { contract, selectedCandidate } = this.state;

  await this.authenticate();

  const accounts = await window.web3.eth.getAccounts();
  const account = accounts[0];

  const voteCount = await contract.methods.getVotes(selectedCandidate).call();

  await contract.methods.vote(selectedCandidate).send({
    from: account,
    gas: 200000,
  });

  this.setState({
    voteCounts: {
      ...this.state.voteCounts,
      [selectedCandidate]: parseInt(voteCount) + 1,
    },
  });
};
```

It is known as .Prior to the vote transaction, authenticate() and watch for the dappkitResponse to be returned. We then utilize contract.methods to conduct the vote transaction after retrieving the user's account.vote(selectedCandidate).send(). Finally, we update the state's vote total.

In order to retrieve the vote totals for each candidate, we also need to alter our getVoteCounts function:

```javascript
async getVoteCounts() {
    const { contract, candidates } = this.state;
    const voteCounts = {};
    for (let i = 0; i < candidates.length; i++) {
      const candidate = candidates[i];
      const voteCount = await contract.methods.getVotes(candidate).call();
      voteCounts[candidate] = parseInt(voteCount);
    }
    this.setState({ voteCounts });
  }
```

We invoke contract.methods for each candidate in a loop.getVotes(candidate).We use call() to get the number of votes cast, then we save it in the voteCounts object in the state.

## 10: Display Vote Counts

To display the vote totals for each contender, we must lastly change the Voting.js file. We'll introduce a new function, renderVoteCounts, which returns a list of candidates together with the total number of votes they've received:

```javascript
renderVoteCounts() {
    const { candidates, voteCounts } = this.state;
    const items = [];
    for (let i = 0; i < candidates.length; i++) {
      const candidate = candidates[i];
      const voteCount = voteCounts[candidate];
      items.push(
        <li key={candidate}>
          {candidate}: {voteCount}
        </li>
      );
    }
    return (
      <ul>{items}</ul>
    );
  }
```

We also need to modify our render function to call this.renderVoteCounts():

```javascript
render() {
    const { candidates, loading } = this.state;
    if (loading) {
      return <div>Loading...</div>;
    }
    return (
      <div className="container">
        <h1>Vote for your favorite candidate</h1>
        <form onSubmit>
```

## Conclusion

Finally, utilizing Solidity and the Truffle framework, we were able to effectively develop a straightforward voting DApp for the Celo network. In order to get started, we first set up our development environment and used Solidity to build a smart contract. The contract was then deployed on the Celo network once we configured our network parameters.

Additionally, we used React to create the user interface and the web3 library to communicate with our smart contract. Finally, we put our DApp to the test by casting a ballot for a candidate and showing the results.

This tutorial gives you a fundamental grasp of how to use Solidity and Truffle to build a DApp for the Celo network. With this framework, you may construct more intricate DApps that communicate with the Celo blockchain and make use of the capabilities of the platform to develop ground-breaking solutions.

Regardless of their background or location, developers may contribute to the development of a more open and accessible financial system by building DApps on the Celo network. Decentralized applications that may assist solve real-world problems are becoming more and more necessary as blockchain technology is being adopted more widely. The Celo network offers developers a strong platform on which to build these applications and advance global financial inclusion.
