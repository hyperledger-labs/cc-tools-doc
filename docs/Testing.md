# Testing

**GoLedger CC-Tools** possui diferente formas de testar o código-fonte em desenvolvimento.

Para verificação de sintaxe da linguagem de programação **GoLang** sugere o uso do seguinte comando:

```sh
cd chaincode \
go vet
```

Após o código corretamente instanciado ou atualizado, pode-se verificar os logs diretamentes nos containers de execução do **chaincode** nos **peers**. Esses containers podems ser identificados por começarem com `dev`. 

```sh
docker logs dev-peer0.org1.example.com-cc-tools-demo-0.1
```

## Using cc-webclient

Para realizar os testes e integrações de forma satisfatório, sugere-se utilizar a ferramenta **cc-webclient**. Terminanda as etapas sugere realizar os seguintes comandos para a criação de 3 aplicações para conectarem com as **org1, org2 e org3**.

```sh
./run-cc-web.sh 8080 & \
./run-cc-web.sh 8090 & \
./run-cc-web.sh 8100 &
```

## Configuring rest-server and cc-webclient

Após a execução dos containers **cc-webclient**, as aplicações podem ser acessadas diretamente pelas portas definidas no script. Ex: 8080, 8090, 8100

Após o acesso ao cc-webclient pelo browser, a configuração para acesso ao Rest server, pode ser feito clicando no ícone de ferramenta.

![Config](/img/header-cc-webclient.png)

Os acessos são feitos com as seguintes configurações:

- `org1`: **http://localhost:80**
- `org2`: **http://localhost:980**
- `org3`: **http://localhost:1080**

![Config](/img/config-cc-webclient.png)

A aplicação **cc-webclient** possui uma barra lateral para mostrar os **assets** disponíveis assim como as **transactions** cadastradas no chaincode.

![Bar](/img/bar-cc-webclient.png)

## Endpoint usage

A utilização do Rest server é descrita através os botões **CURL**. 

![CURL](/img/curl-cc-webclient.png)

Para cada tela, pode-se verificar uma chamada utilizando a aplicação `curl`. 

Por exemplo, na tela de criação de ativo:

![CURL](/img/curl-create-cc-webclient.png)

## List, create, edit, delete or history an asset

Para **listar**, basta selecionar um asset na barra lateral

![List Asset](/img/list-cc-webclient.png)

Selecionando o botão `CREATE` na janela de listagem de um asset, uma tela de **criação** de ativo vai aparecer.

Por exemplo, para o asset **Person**

![Create Asset](/img/create-asset-cc-webclient.png)

A tela de **edição** é acessada selecionando o ícone de editar na janela de listagem de um asset.

A **deleção** de um asset é requisitada selecionando o ícone de apagar na janela de listagem de um asset.

O **histórico** de um asset (todas as modificações registradas no ledger) pode ser visualizada selecionando o ícone de histórico na janela de um asset.

## Executing an transaction

A execução de uma **transação** pode ser realizada selecionando a transação na aba lateral.

Por exemplo, para a transação **UpdateBookTenant**

![Transaction](/img/transaction-cc-webclient.png)




