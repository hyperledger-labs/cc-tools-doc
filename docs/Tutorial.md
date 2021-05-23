# Writing Your First Application

Neste tutorial, iremos instalar e utilizar um contrato inteligente (chaincode).

É necessário estar com o ambiente configurado para poder realizar esse tutorial. Detalhes em [Getting Started](get%20Started)

Veremos como a biblioteca GoLedger CC-Tools funciona, utilizando os artefatos disponibilizados pela GoLedger

- Rest-server integrada
- Api de atualização Hyperledger Fabric
- Aplicação Web de desenvolvimento (cc-web)

O contrato inteligente do respositório cc-tool-demo é ideal para os iniciantes ao desenvolvimento na tecnologia Hyperledger Fabric. É um ótimo ponto de partida para entender uma blockchain do Hyperledger Fabric e facilitar a jornada para o deenvolvimento de aplicações utilizando esse framework. 

Você aprenderá como escrever um aplicativo e um contrato inteligente (chaincode) para consultar e atualizar um livro razão, e como se conectar com o Blockchain através de uma API pronta para uso.

---
**<span style="color:blue">NOTE</span>**

É necessário conhecer a linguagem de programação *GoLang* para realizar esse tutorial.

---

## Network Details

A rede a ser criada terá a seguinte configuração:

- 1 chaincode (cc-tools-demo)
- 3 orgs (org1, org2, org3)
- 3 clients (rest-server para as org1, org2 e org3)

## Chaincode Details

O chaincode disponibilizado possui as seguintes caraterísticas:

- 3 assets
- 1 private data
- 3 transações

## MSP Configuration

As redes Hyperledger Fabric necessitam de correta configuração dos certificados digitais x.509 agrupados em elementos demoninado **Membership Service Provider** ou simplesmente chamados **<span style="color:blue">MSP</span>**

Os certificados são gerados e adminstrados por autoridades certificadora (**CA**) utilizando a plataforma **Hyperledger Fabric CA**. Cada organização (org1, org2 e org3) será representada por uma **CA**

---

**<span style="color:blue">NOTE</span>**

Não é objetivo desse tutorial ensinar os conceitos de MSP ou do **Hyperledger Fabric CA**

---

A geração dos certificados e MSP necessários para geração da rede é realizado através do script abaixo a ser executado na pasta fabric:

```sh
cd fabric \
rm -rf crypto-config channel-artifacts ca && \
./startDev.sh generate && \
cd ..
```

Esse script ira subir 3 containers do **Hyperledger Fabric CA**
- ca.org1.example.com
- ca.org2.example.com
- ca.org3.example.com

Ao final 3 pastas serão criadas com os artefatos criptográficos necessários para a geração da rede

```
.
├── fabric
    └── ca    
    └── crypto-config
    └── channel-artifacts
```

## Vendoring

Tanto o chaincode em GoLang quanto a rest-server em NodeJs precisam ser vendorados (baixar os pacotes de dependências) para poderem funcionar.

Para baixar as dependências do chaincode deve ser executado o seguinte script:

```sh
cd chaincode && \
go mod vendor && \
cd ..
```

Para baixar as dependências do rest-server deve ser executado o seguinte script

```sh
cd rest-server && \
npm install && \
cd ..
```

## Building your GoLedger CC-Tools network

Depois da instalacão do ambiente, geração dos certificados e vendoração dos pacotes, a rede está pronta para ser instanciada.

Esse processo se dá com o seguinte script

```sh
./startDev.sh
```

Esse script irá executar as seguintes tarefas:

- Criar os containers para o nós do Blockchain (peers e ordereres no **Hyperledger Fabric**) para as 3 orgs (org1, org2, org3) com a correta configuração criptográfica para cada um deles.
- Criar o ledger para registro e permissionamento - **Hyperledger Fabric Channel**
- Entrada das organizações no channel (processo de Join no **Hyperledger Fabric**)
- Instalação do chaincode nos endorsing peers
- Instalação da api de gerenciamento da rede **Hyperledger Fabric**
- Instanciar o chaincode no channel
- Criar os containers para as Rest servers com a correta configuração criptográfica para respresentarem um **Hyperledger Fabric Client** para cada organização.

Ao final do processo, deve ser apresentado as seguintes mensagens

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

Os containers **ccapi** representam os servidores rest para cada organização. Eles podem levar alguns minutos para ficarem online.

Ao final, a seguinte mensagem deve aparecer:

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
---

**<span style="color:blue">NOTE</span>**

Em alguns equipamentos pode aparecer a seguinte mensagem de erro:

Error: Failed to load gRPC binary module because it was not installed for the current system
Expected directory: node-v57-linux-x64-glibc
Found: [node-v64-linux-x64-glibc]

Nesse caso deve-se rodar o seguinte script

```sh
cd rest-server
npm rebuild --target=8.1.0 --target_platform=linux --target_arch=x64 --target_libc=glibc --update-binary
cd ..
```

---
