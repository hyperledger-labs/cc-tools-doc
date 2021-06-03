# Getting Started

**GoLedger CC-Tools** library was developed to be used in Linux operating system.

In general we use the Ubuntu 18+ distribution, however the library is compatible with other environments, but some adjustments may be necessary.

## Download demo code

To learn how to use **GoLedger CC-Tools library**, download the demo code repository available on **Github**.

```sh
cd $HOME \
git clone https://github.com/goledgerdev/cc-tools-demo.git \
cd cc-tools-demo
```

## Folders distribution

```
. { cc-tools-demo root folder }
|
├── chaincode { Smart contract code (GoLang) }
|   └── assettypes { Asset definitions }
|   └── txdefs { Blockchain transactions }
|   └── datatypes { Custom property datatypes }
|
├── rest-server { Rest API code (NodeJs) }
|
├── fabric { Hyperledger Fabric artifacts }
```

This repository has the necessary elements to use the main functions of the library.

## Enviroment configuration

The following systems, platforms and languages ​​must be installed:

- Docker 19+
- GCC
- GoLang 1.14+
- NodeJs 10+
- Hyperledger Fabric 1.4 (Hyperledger Fabric 2.x ainda em roadmap)

If you are using **Linux Ubuntu**, run the following command, which will download and install the systems above before starting development.

The repository has a configuration script.

```sh
./installPreReqUbuntu.sh
```
At the end, the following success message should appear.

```sh
Enviroment configured
```