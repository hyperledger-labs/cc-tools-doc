# Tutorials

## Writing Your First Application

This tutorial will show how to install and use a smart contract (chaincode).

You must already have configured the environment to be able to perform this tutorial. Details on [Getting Started](getting-started.md)

We'll see how the GoLedger CC-Tools library works, using the artifacts provided by GoLedger

- CCAPI integration
- Hyperledger Fabric Update API
- Development web application (cc-web)

The `cc-tools-demo` repository smart contract is ideal for beginners to Hyperledger Fabric technology development. It's a great starting point to understand a Hyperledger Fabric blockchain and ease the journey of application development using this framework.

You'll learn how to write an application and a smart contract (chaincode) to query and update a ledger, and how to connect to the Blockchain through a ready-to-use API.

---
**<span style="color:blue">NOTE</span>**

You need to know *GoLang* programming language in order to complete this tutorial.

---

### Network Details

The standard network to be created will have the following configuration:

- 1 chaincode (cc-tools-demo)
- 3 orgs (org1, org2, org3)
- 3 clients (rest servers for org1, org2 and org3)

There is also an option to create a smaller network with fewer entities, with the following configuration:

- 1 chaincode (cc-tools-demo)
- 1 org (org)
- 1 client (rest server for org)

### Chaincode Details

The chaincode provided in `cc-tools-demo` has the following characteristics:

- 3 assets
- 1 private data
- 4 transactions

### Vendoring

Both the **chaincode** and the **CCAPI**, in Golang, need to be vendored (dependency packages download) in order to properly work.

To download chaincode dependencies, run the following script:

```sh
cd chaincode && \
go mod vendor && \
cd ..
```

To download the CCAPI dependencies, run the following script:

```sh
cd ccapi && \
go mod vendor && \
cd ..
```

### Building your GoLedger CC-Tools network

After installing the environment and vendoring the packages, the network is ready to be instantiated.

This process takes place with the following script

```sh
./startDev.sh
```

This script will perform the following tasks:

- Clear unused docker images and volumes
- Generate the MSP certificates for each org (visit the [Hyperledger Fabric Docs](https://hyperledger-fabric.readthedocs.io/) to learn more).
- Create the Blockchain node containers (**Hyperledger Fabric** peers and orderers) for 3 orgs (org1, org2, org3) with the correct cryptographic configuration for each one of them.
- Create the ledger for registration and permission - **Hyperledger Fabric Channel**
- Organizations entries in the channel (Join process in **Hyperledger Fabric**)
- Chaincode installation inside endorsing peers
- Installation of network management api **Hyperledger Fabric**
- Chaincode instantiation on the channel
- Rest servers containers deployment with the correct cryptographic configuration, representing a **Hyperledger Fabric Client** for each organization.

At the end of the process, the following messages should be displayed:

```sh
Chaincode definition committed on channel 'mainchannel'

...

Query chaincode definition successful on peer0.org1 on channel 'mainchannel'

...

Query chaincode definition successful on peer0.org2 on channel 'mainchannel'

...

Query chaincode definition successful on peer0.org3 on channel 'mainchannel'
Chaincode initialization is not required
[+] Running 3/3
 ⠿ Container ccapi.org2.example.com  Started                        0.3s
 ⠿ Container ccapi.org1.example.com  Started                        0.5s
 ⠿ Container ccapi.org3.example.com  Started                        0.5s
```

The **ccapi** containers represent the rest servers for each organization. They may take a few minutes to come online.

At the end, the following message should appear:

```sh
docker logs ccapi.org1.example.com 

2023/07/27 14:00:53 Listening on port 80
2023/07/27 14:00:53 sdk created from './config/configsdk-org1.yaml'
2023/07/27 14:00:53 sdk created from './config/configsdk-org1.yaml'
Listening on port 80
```

To instantiate the minimal network using only one organization, use the following command:

```sh
./startDev.sh -n 1
```

### Upgrading your chaincode

Upgrading a smart contract can be done easily in chaincodes that use the **GoLedger CC-Tools** library.

First, you must check the syntax of the modified code.

```sh
cd chaincode \
go vet \
cd ..
```

After validation, the chaincode update is done with the `upgradeCC.sh` script, which takes the chaincode version and sequence as arguments.

---

**<span style="color:blue">NOTE</span>**

A chaincode must always be updated with a version different from all previous versions and the sequence should be sequential from the one before.

---

Example:

```sh
./upgradeCC.sh 0.2 2
``` 

Or if you are using the minimal network:

```sh
./upgradeCC.sh 0.2 2 -n 1
``` 

### Reloading CCAPI

If any changes were made to the CCAPI code you must reload it for the changes to take effect. 

First, check the syntax of the modified code.

```sh
cd ccapi \
go vet \
cd ..
```

After validation, the `reloadCCAPI.sh` script can be used to reload the rest server.

```sh
./reloadCCAPI.sh
``` 

Or if you are using the minimal network:

```sh
./reloadCCAPI.sh -n 1
``` 

### Renaming your project

After testing cc-tools-demo and instantiating your first fabric network, you are ready to start developing your first chaincode.

To use your own project name, instead of `cc-tools-demo`, the `renameProject.sh` script is availiable to rename all mentions to it on the project folder. The script takes the new name as an argument.

Example: 

```sh
./renameProject.sh my-first-chaincode
``` 
