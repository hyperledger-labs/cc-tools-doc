# External Tools

A biblioteca **GoLedger CC-Tools** possui uma gama de ferramentas externas que ajudam imensamente na jornada de desenvolvimento de um Blockchain permissionado baseado em **Hyperledger Fabric**

- `cc-tools-demo` - repositório público com exemplo de ativos e transações utilizando a biblioteca **GoLedger CC-Tools**
- `cc-webclient` - imagem docker com aplicação Web para interconexão com a Rest Api.
- `GoFabric` - plataforma de orquestração de redes para ambientes de produção. [https://gofabric.io](https://gofabric.io)

## cc-tools-demo

Repositório aberto com exemplos e transações.

O repositório pode ser acessado da seguinte forma:

```sh
git clone https://github.com/goledgerdev/cc-tools-demo.git
```

## cc-web-client

Imagem docker com aplicação web para testes e auxílio de integrações com sistemas legados.

O container pode ser instanciado da seguinte forma:

```sh
docker run -p 0.0.0.0:8080:80/tcp --name cc-webclient goledger/cc-webclient:latest
```

## GoFabric

**GoFabric** é uma plataforma de orquestração de redes **Hyperledger Fabric** totalmente compatível com a biblioteca **GoLedger CC-Tools**

A plataforma pode ser acessada no seguinte link: [https://gofabric.io](https://gofabric.io) e possui as seguintes funções:

- Deployment de redes ambiente de nuvem (**AWS, IBM, Azure, etc**) ou **on-premise**
- Instanciar chaincodes (**compatibilidade com CC-Tools**)
- Atualizar chaincodes
- Adicionar ou remover **peers**
- Adicionar **orderers**
- Adicionar **orgs**
- Instanciar ou atualizar **Rest Servers**
- Geração automática de código **GoLedger Templates**

![Config](img/gofabric.png)

