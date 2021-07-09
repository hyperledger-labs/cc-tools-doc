# Tutorials

## Writing Your First Application

This tutorial will show how to install and use a smart contract (chaincode).

You must already have configured the environment to be able to perform this tutorial. Details on [Getting Started](getting-started.md)

We'll see how the GoLedger CC-Tools library works, using the artifacts provided by GoLedger

- Rest-server integration
- Hyperledger Fabric Update API
- Development web application (cc-web)

The `cc-tools-demo` repository smart contract is ideal for beginners to Hyperledger Fabric technology development. It's a great starting point to understand a Hyperledger Fabric blockchain and ease the journey of application development using this framework.

You'll learn how to write an application and a smart contract (chaincode) to query and update a ledger, and how to connect to the Blockchain through a ready-to-use API.

---
**<span style="color:blue">NOTE</span>**

You need to know *GoLang* programming language in order to complete this tutorial.

---

### Network Details

The network to be created will have the following configuration:

- 1 chaincode (cc-tools-demo)
- 3 orgs (org1, org2, org3)
- 3 clients (rest-server para as org1, org2 e org3)

### Chaincode Details

The chaincode provided in `cc-tools-demo` has the following characteristics:

- 3 assets
- 1 private data
- 3 transactions

### MSP Configuration

Hyperledger Fabric networks require the correct configuration of x.509 digital certificates grouped in elements called **Membership Service Provider** or simply called **<span style="color:blue">MSP</span>**

Certificates are generated and managed by certification authorities (**CA**) using the **Hyperledger Fabric CA** platform. Each organization **(org1, org2 and org3)** will be represented by a unique **CA**

---

**<span style="color:blue">NOTE</span>**

It is not the purpose of this tutorial to teach the concepts of MSP or **Hyperledger Fabric CA**, for that check the official [HF Docs](https://hyperledger-fabric.readthedocs.io/).

---

The generation of certificates and MSP necessary for network generation is performed through the script below to be executed in the `./fabric` folder:


```sh
cd fabric \
rm -rf crypto-config channel-artifacts ca && \
./startDev.sh generate && \
cd ..
```

This will deploy 3 containers of **Hyperledger Fabric CA**

- ca.org1.example.com
- ca.org2.example.com
- ca.org3.example.com

After the script finishes, 3 folders will be created with the cryptographic artifacts necessary to generate the network:

```
.
├── fabric
    └── ca    
    └── crypto-config
    └── channel-artifacts
```

### Vendoring

Both the **chaincode** in GoLang and the **rest-server** in Node.js need to be vendored (dependency packages download) in order to work.

To download chaincode dependencies, run the following script:

```sh
cd chaincode && \
go mod vendor && \
cd ..
```

To download the rest-server dependencies, run the following script:

```sh
cd rest-server && \
./npmInstall.sh && \
cd ..
```

### Building your GoLedger CC-Tools network

After installing the environment, generating the certificates and vendoring the packages, the network is ready to be instantiated.

This process takes place with the following script

```sh
./startDev.sh
```

This script will perform the following tasks:

- Create the Blockchain node containers (**Hyperledger Fabric** peers and orderers) for 3 orgs (org1, org2, org3) with the correct cryptographic configuration for each one of them.
- Create the ledger for registration and permission - **Hyperledger Fabric Channel**
- Organizations entries in the channel (Join process in **Hyperledger Fabric**)
- Chaincode installation inside endorsing peers
- Installation of network management api **Hyperledger Fabric**
- Chaincode instantiation on the channel
- Rest servers containers deployment with the correct cryptographic configuration, representing a **Hyperledger Fabric Client** for each organization.

At the end of the process, the following messages should be displayed:

```sh
Create channel - mainchannel
{"status":"SUCCESS","created":true}

Join org1 to channel
{"status":"SUCCESS"}

Join org2 to channel
{"status":"SUCCESS"}

Join org3 to channel
{"status":"SUCCESS"}

Update anchor peers on org1
{"status":"SUCCESS"}

Update anchor peers on org2
{"status":"SUCCESS"}

Update anchor peers on org3
{"status":"SUCCESS"}

Install network chaincode on org1
{"installed":true,"version":"1.0"}

Install network chaincode on org2
{"installed":true,"version":"1.0"}

Install network chaincode on org3
{"installed":true,"version":"1.0"}

Instantiate network chaincode
{"started":true}

Install chaincode on org1

Install chaincode on org2

Install chaincode on org3

Instantiate chaincode
{"started":true}

Network cc-tools-demo-net is external, skipping
Creating ccapi.org1.example.com ... done
Creating ccapi.org2.example.com ... done
Creating ccapi.org3.example.com ... done
```

The **ccapi** containers represent the rest servers for each organization. They may take a few minutes to come online.

At the end, the following message should appear:

```sh
docker logs ccapi.org1.example.com 

> cc-tools-demoapi@1.0.0 start /rest-server
> gulp default; gulp start

[23:12:58] Using gulpfile /rest-server/gulpfile.js
[23:12:58] Starting 'default'...
[23:12:58] Starting 'assets'...
[23:12:59] Finished 'default' after 49 ms
[23:12:59] Finished 'assets' after 51 ms
[23:13:40] Using gulpfile /rest-server/gulpfile.js
[23:13:40] Starting 'start'...
[23:13:41] [nodemon] 1.19.0
[23:13:41] [nodemon] to restart at any time, enter `rs`
[23:13:41] [nodemon] watching: /rest-server/src/**/*
[23:13:41] [nodemon] starting `node dist/`
Listening on port 80
```

### Updating your chaincode

Updating a smart contract can be done easily in chaincodes that use the **GoLedger CC-Tools** library.

First, you must check the syntax of the modified code.

```sh
cd chaincode \
go vet \
cd ..
```

After validation, the chaincode update is done with the `upgradeCC.sh` script, which takes the chaincode version as argument.

---

**<span style="color:blue">NOTE</span>**

A chaincode must always be updated with a version different from all previous versions.

---

Example:

```sh
./upgradeCC.sh 0.2
``` 

