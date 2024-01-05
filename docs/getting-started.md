# Getting Started



**GoLedger CC-Tools** library was developed to be used in Linux operating system.

In general we use the Ubuntu 18+ distribution, however the library is compatible with other environments, but some adjustments may be necessary.

## Download and setup 

To learn how to use **GoLedger CC-Tools library**, download the demo code repository available on **GitHub**.

```sh
cd $HOME \
git clone https://github.com/hyperledger-labs/cc-tools-demo.git \
cd cc-tools-demo
```

### Folder distribution

```
. // cc-tools-demo root folder 
|          
├── ccapi               // Rest API code (Golang Gin server) 
|
├── chaincode           // Smart contract code (GoLang) 
|   └── assettypes      // Asset definitions
|   └── eventtypes      // Asset definitions 
|   └── txdefs          // Blockchain transactions 
|   └── datatypes       // Custom property datatypes 
|
├── fabric              // Hyperledger Fabric artifacts 
```

These are all the necessary elements to use the main functions of the library.

## Enviroment configuration

The following systems, platforms and languages ​​must be installed:

- Docker 19+
- GCC
- GoLang 1.14+
- NodeJs 10+
- Hyperledger Fabric 2.5

If you are using **Linux Ubuntu**, run the following command from the root directory. This will download and install the systems above before starting development.

```sh
./scripts/installPreReqUbuntu.sh
```

At the end, the following success message should appear.

```sh
Enviroment configured
```

Now you're ready to [Write Your First Application](tutorials.md).