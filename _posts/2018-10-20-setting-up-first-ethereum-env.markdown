---
layout: post
permalink: /tutorials/setting-up-first-ethereum-dapp-env
title: "Getting Set Up for your First Ethereum Ðapp Build"
description: ""
date: 2018-10-20 09:30
tags:
- bancor-network
- ethereum-applications
- dApps
- bancor-api
- cryptocurrency
- crypto-development-setup
---

**TL;DR:**  After covering the basics of getting set up on the Ethereum Blockchain, we will cover some of the most useful &amp; popular tools and resources for Ðapp development.

Getting Set Up for your First Ethereum Ðapp Build
==============================

It can be a fairly daunting process when looking to start-off developing Crypto applications.  I know for sure, I didn't really know where to start or which resources to look at.  Where do we begin?  Do we need a wallet somewhere?  Are we using Ethereum smart contracts?  And if so - do we need to learn [*Solidity*](https://blockgeeks.com/guides/solidity/)?

All of these questions are common, and will be answered, alongside a few more in this tutorial.  The aim of this article is to give you all the information and resources needed to get started building your first Ethereum Ðapp.  Handily, there are now many tools available to ease development, and the vast majority are not only extremely useful, but also highly impressive.

After covering the basics of getting set up on the Ethereum Blockchain, we will cover some of the most useful &amp; popular tools and resources for Ðapp development.  Before we cover app set up, let's take a brief look at the technologies involved:


### Ethereum

Ethereum is an Open Source, Decentralised technology and network of nodes which features ***Smart Contract*** functionality.  Ethereum allows anyone to build and use decentralised applications that run on Blockchain technology, offering a much higher level of flexibility than the Bitcoin and other Blockchains.

### Smart Contracts

Smart Contracts are scripts of code running on the Blockchain that contain sets of rules for parties of that contract to interact with one another.  The Smart Contract rules allow for automatic enforcement when specified conditions are met.  The state of these contracts is stored on the Blockchain allowing for immutability and accountability.

### Ethereum Virtual Machine (EVM)

The EVM is the runtime environment used for Ethereum's smart contracts.  It can be used to automatically conduct transactions or perform specific actions on the Ethereum blockchain.  For security purposes, the EVM ensures programs do not have access to each other’s state, ensuring communication can be established without any potential interference.

### Ethereum APIs &amp; Web3 Technologies

With the Ethereum Blockchain running on a network of nodes, we need a way of interacting with it from our web applications / browsers.  For this, we have the Web3 APIs.  Web3.js talks to The Ethereum Blockchain through JSON RPC.  It is a collection of libraries that allow us to perform actions like send Ether from one account to another, read and write data from smart contracts, create smart contracts, and much more.

The image below shows the Web3 stack denoting users connecting in-browser to an Ethereum application that interfaces with data on the Blockchain.  Each client (browser) communicates with it’s own instance of the application, as there is no central server to which all clients connect.

![Web3 Stack](/assets/img/1-WNbOOqsXIwZgtx5kw1Xo3g.png) 

<br/>
# Ethereum Installation &amp; Setup

Now that we've covered the main technologies involved in Ðapp development, let's dive into installing Ethereum and the related technologies we need.  This tutorial is going to assume an operating system of ***Ubuntu (16.04).***

The first thing we need to do is add the Ethereum repository, and ensure we are up-to-date by running the following:

```bash
$ sudo add-apt-repository -y ppa:ethereum/ethereum
$ sudo apt-get update
$ sudo apt-get -y upgrade
```

The next thing we need to do is install Go Ethereum.  Go Ethereum is one of the three original implementations (along with C++ and Python) of the Ethereum protocol.

```bash
$ sudo apt-get install ethereum
```

We can use Geth to deploy live contracts and interact with the Ethereum Blockchain, but for testing purposes, we are better off installing a Blockchain test environment.  For this, we will use [Ganache](https://truffleframework.com/ganache).  You can install *Ganache CLI* by running:


```bash
$ npm install -g ganache-cli
```

Once you have installed the Ganache CLI, you may start the program by running:

```bash
$ ganache-cli
```

Starting Ganache will create 10 accounts each containing 100 ETH that you can utilise for local testing!

![Ganache CLI Accounts](/assets/img/ganache-cli-accounts.png) 

Now that we have our Ethereum Blockchain and test accounts running, we are all set up to create and deploy our DApp.  We will be using ***Truffle*** to generate our DApps and provide the frameworks we need.  Go ahead and run:

```bash
$ npm install truffle -g
```

Truffle provides an incredibly useful feature called ***'Boxes'***, that contain everything you need to get started developing both the Ethereum underpinnings and web frontend to your DApps.  You can find a list of [Boxes here.](https://truffleframework.com/boxes)

If you wish to initialise an empty project with no sample Contracts in, you can do so using the `truffle init` command:

```bash
$ mkdir dappy
$ cd dappy

$ truffle init
```

The generated project structure looks like the following:

 - ***contracts/*** Directory for Solidity contracts
 - ***migrations/*** Directory for scriptable deployment files
 - ***test/*** Directory for test files for testing your application and contracts
 - ***truffle.js*** Truffle configuration file

For your first application, why not check out the sample ***MetaCoin*** box which will allow you get a taster for smart contracts and DApp development / deployment.  You can get the required files by running:

```bash
$ truffle unbox metacoin
```

Now that we have Ganache running a test Blockchain with accounts for us to utilise, and we know how to generate DApp projects, let's take a look at interacting with the test Blockchain and accounts from our browser.  For this, we will use the popular [https://metamask.io](https://metamask.io).

Head to the Metamask site above, and install the browser plugin for your chosen web browser.  Once installed, we can set up Metamask to connect to our Ganache test network.  Click on the Metamask plugin icon, and then on the ***Restore Vault*** link.

Copy the Mnenomic phrase provided when you started the Ganache CLI client.  i.e:

<center><img src="/assets/img/metamask-seed.png" height="400px" /></center>

Once you have done that, we can connect Metamask to the blockchain created by Ganache.  When we started the Ganache CLI, it told us:

> "Listening on 127.0.0.1:8545"

We can now take this URL to connect Metamask.  Click on the ***"Main Network"*** link in the top-left corner.  In the box titled "New RPC URL" enter http://127.0.0.1:8545 and click Save.

<center><img src="/assets/img/metamask-networkmenu.png" height="400px" /></center>

Now that we have connected Metamask to our Ganache Blockchain, we can see the test accounts generated, and their Ether contents.  The first account should have less than the others because that account supplies the gas for smart contract deployment. Since you've deployed your smart contract to the network, this account paid for it:

<center><img src="/assets/img/metamask-account1.png" height="400px" /></center>

And with that, we have an Ethereum Blockchain running on our machine, we with a full development environment set up and ready to go!

There is a fantastic official overview exploring Ethereum development in detail, that [can be found here.](https://github.com/ethereum/wiki/wiki/Ethereum-Development-Tutorial)


<br/>
## Aside:  The Bancor Network

Bancor offers a decentralized liquidity network that provides users with a simple, low-cost way to buy and sell tokens. Bancor’s open-source protocol empowers tokens with built-in convertibility directly through their smart contracts, allowing integrated tokens to be instantly converted for one another, without needing to match buyers and sellers in an exchange. 

The Bancor Wallet enables automated token conversions directly from within the wallet, at prices that are more predictable than exchanges and resistant to manipulation.