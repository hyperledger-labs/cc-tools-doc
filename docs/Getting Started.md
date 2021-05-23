# Getting Started

O biblioteca GoLedger CC-Tools foi desenvolvida para ser utilizando em sistema operacional Linux.

Em geral utilizamos a distribuiçã Ubuntu 18+, porém a biblioteca é compatível com outros ambientes, mas alguns ajustes podem ser necessários.

## Download demo code

Para aprender a utilizar a biblioteca GoLedger CC-Tools baixe o respositório do código de demonstração disponível no Github.

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


Esse repositório possui os elementos necessários para utilizar as principais funções a biblioteca.

## Enviroment configuration

Os seguintes sistemas, plataformas e linguagens devem estar instaladas:

- Docker 19+
- GCC
- GoLang 1.14+
- NodeJs 10+
- Hyperledger Fabric 1.4 (Hyperledger Fabric 2.x ainda em roadmap)

Para Linux Ubuntu, o execute o seguinte comando, que fará o download e instalação dos sistemas acima antes de iniciar o desenvolvimento.

O repositório possui um script de configuração.

```sh
./installPreReqUbuntu.sh
```

Ao final da execução, deve aparecer a seguinte mensagem de sucesso.
```sh
Enviroment configured
```