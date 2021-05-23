# Assets

Os assets (ou ativos do Blockchain) representam as informações que serão utilizadas pelo ledger do Blockchain (**Hyperledger Fabric Channel**)

A grosso modo, um asset pode ser comparado a uma tabela de um banco de dados relacional.

E tal como nos bancos dados, os assets são formados por propriedades, sendo que algumas delas podem fazer parte de um conjunto de chaves desse ativo. Cada propriedade pode ter um tipo de dado específico, como string, number, boolean, etc.

Um asset também pode ser um **Hyperledger Fabric Private Data**, na qual o conteúdo das informações são gravados em um bases externas ao channel e apenas um subconjunto das organizações recebem esses dados, configurando capacidade de leitura.

Para o repositório cc-tools-demo, 4 assets (sendo que um deles é um private data)

- Person
- Book
- Library
- Secret (private data)

Para uso da biblioteca GoLedger CC-Tools, a definição dos assets é feito na pasta **chaincode/assettypes**

A seguir está lista de arquivos

    chaincode/
        assettypes/     # asset foldgers
            person.go   # definition of a person
            book.go     # definition of a book
            library.go  # definition of a library
            secret.go   # definitions of secret (private data)
    assetTypeList.go    # lista of assets instantiated

