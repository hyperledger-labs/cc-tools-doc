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
    assetTypeList.go    # list of assets instantiated

## Asset definition

A construção e definição de um asset se faz com a criação de um arquivo dentro da pasta **chaincode/assettypes**

Um asset possui os seguintes campos

- `Tag`: campo string para definir o nome do asset referenciado internamento pelo código e pela Rest Api. Não pode ter espaços ou caracteres especiais.
- `Label`: campo string para definir o rótulo para ser utilizado nas aplicações externas. Texto livre.
- `Description`: campo string de descrição do asset para ser utilizado nas aplicações externas. Texto livre.
- `Props`: propriedades do asset. Possui campos proprios.
- `Readers`: utilizado para definir as organizações do private data (precisa de configuração adicional no arquivo **collections.json**)


## Property definition

Um asset possui um conjunto de propriedades (Props) para arquitetar o ativo. 

Uma propriedade possui os seguintes campos:

- `IsKey`: idenfitica se a propriedade faz parte das chaves do asset. Campo booleano.
- `Required`: identifica se a propriedade é obrigatória. Campo booleano.
- `ReadOnly`: identifica se a propriedade não pode mais ser modificada depois de criadas. Campo booleano.
- `DefaultValue`: valor padrão para a propriedade.
- `Writers`: utilizado para quais organizações podem criar ou alterar essa propriedade. Se a propriedade for chave (campo isKey: true) então todo o asset só pode ser criado ou alterado pela organização. Lista de strings.
- `DataType`: tipo da propriedade. CC-Tools possui os seguintes tipos padrão: **string, number, datetime e boolean**. Tipos customizados podem ser definidos na pasta **chaincode/datatypes**.
- `Validade`: função de validade da propriedade. Sugere-se utilizar apenas para validações simples, funções mais complexas deve-se utilizar os datatypes customizados.

## Asset examples

O repositório cc-tools-demo traz alguns exemplos:

    chaincode/
        assettypes/     # asset foldgers
            person.go   # definition of a person
            book.go     # definition of a book
            library.go  # definition of a library
            secret.go   # definitions of secret (private data)
    assetTypeList.go    # list of assets instantiated

Além dos arquivos de cada asset, deve-se cadastrar os assets que podem ser utilizado pela biblioteca Goledger CC-Tools no arquivo **assetTypeList.go**

A definição do asset **Person** é a seguinte:

    var Person = assets.AssetType{
    	Tag:         "person",
    	Label:       "Person",
    	Description: "Personal data of someone",

    	Props: []assets.AssetProp{
	    	{
		    	// Primary key
			    Required: true,
			    IsKey:    true,
			    Tag:      "id",
			    Label:    "CPF (Brazilian ID)",
			    DataType: "cpf",               // Datatypes are identified at datatypes folder
			    Writers:  []string{`org1MSP`}, // This means only org1 can create the asset (others can edit)
		    },
		    {
    			// Mandatory property
			    Required: true,
			    Tag:      "name",
			    Label:    "Asset Name",
			    DataType: "string",
			    // Validate funcion
			    Validate: func(name interface{}) error {
    				nameStr := name.(string)
				    if nameStr == "" {
    					return fmt.Errorf("name must be non-empty")
				    }
				    return nil
			    },
		    },
		    {
    			// Optional property
	    		Tag:      "dateOfBirth",
		    	Label:    "Date of Birth",
			    DataType: "datetime",
		    },
		    {
			    // Property with default value
			    Tag:          "heigth",
			    Label:        "Person's heigth",
			    DefaultValue: 0,
			    DataType:     "number",
		    },
	    },
    }

De acordo com a descrição, o ativo **Person** tem as seguintes caracteríticas:

- Chave primária **cpf** (datatype customizado)
- Apenas a **org1** pode criar ou alterar a propriedade **cpf** e o asset **Person** (porque essa é uma chave)
- Propriedade **name** de tipo **string** é obrigatória, e possui validação automática para impedir strings vazias ("")
- Propriedade **dateofBirth** de tipo **datetime** opcional.
- Propriedade **heigth** de tipo **number**, com valor padrão 0

A definição do asset **Book** é a seguinte:

    var Book = assets.AssetType{
    	Tag:         "book",
    	Label:       "Book",
    	Description: "",

    	Props: []assets.AssetProp{
		    {
			    // Composite Key
			    Required: true,
			    IsKey:    true,
			    Tag:      "title",
			    Label:    "Book Title",
			    DataType: "string",
			    Writers:  []string{`org2MSP`}, // This means only org2 can create the asset (others can edit)
		    },
		    {
    			// Composite Key
			    Required: true,
			    IsKey:    true,
			    Tag:      "author",
			    Label:    "Book Author",
			    DataType: "string",
			    Writers:  []string{`org2MSP`}, // This means only org2 can create the asset (others can edit)
		    },
		    {
    			/// Reference to another asset
			    Tag:      "currentTenant",
			    Label:    "Current Tenant",
			    DataType: "->person",
		    },
		    {
    			// String list
			    Tag:      "genres",
			    Label:    "Genres",
			    DataType: "[]string",
		    },
		    {
    			// Date property
	    		Tag:      "published",
		    	Label:    "Publishment Date",
    			DataType: "datetime",
	    	},
	    },
    }

De acordo com a descrição, o ativo **Book** tem as seguintes caracteríticas:

- Chave composta  **title** e **author**, ambas com tipo **string**
- Apenas a **org2** pode criar ou alterar as propriedades **title** e **author** e o asset **Book** (porque são chaves)
- Propriedade **currentTenant** de tipo **->person** o que representa a referência a um asset **Person**
- Propriedade **genres** de tipo **[]string** que representa um array de strings
- Propriedade **published** de tipo **datetime**

A definição do asset **Library** é a seguinte:

    var Library = assets.AssetType{
    	Tag:         "library",
	    Label:       "Library",
	    Description: "Library as a collection of books",

	    Props: []assets.AssetProp{
    		{
	    		// Primary Key
		    	Required: true,
			    IsKey:    true,
    			Tag:      "name",
	    		Label:    "Library Name",
		    	DataType: "string",
			    Writers:  []string{`org3MSP`}, // This means only org3 can create the asset (others can edit)
    		},
	    	{
		    	// Asset reference list
    			Tag:      "books",
	    		Label:    "Book Collection",
		    	DataType: "[]->book",
    		},
	    	{
		    	// Asset reference list
			    Tag:      "entranceCode",
    			Label:    "Entrance Code for the Library",
	    		DataType: "->secret",
    		},
    	},
    }
De acordo com a descrição, o ativo **Library** tem as seguintes caracteríticas:

- Chave primária **name** de tipo **string**
- Apenas a **org3** pode criar ou alterar a propriedade **name** e o asset **Library** (porque é chave)
- Propriedade **books** de tipo **[]->book** o que representa um array de referências ao asset **Book**
- Propriedade **entranceCode** de tipo **->secret** que representa a referência a um tipo private data

A definição do asset **Secret** é a seguinte:
    var Secret = assets.AssetType{
	    Tag:         "secret",
    	Label:       "Secret",
	    Description: "Secret between Org2 and Org3",

    	Readers: []string{"org2MSP", "org3MSP"},
	    Props: []assets.AssetProp{
    		{
	    		// Primary Key
		    	IsKey:    true,
			    Tag:      "secretName",
			    Label:    "Secret Name",
    			DataType: "string",
			    Writers:  []string{`org2MSP`}, // This means only org2 can create the asset (org3 can edit)
		    },
    		{
	    		// Mandatory Property
		    	Required: true,
			    Tag:      "secret",
			    Label:    "Secret",
    			DataType: "string",
	    	},
	    },
    }
De acordo com a descrição, o ativo **Secret** tem as características de privacidade (**Hyperledger Fabric Private Data**). Para esse ativo funcionar corretamenta, deve-se configurar o arquivo **collections.json**

Não iremos descrever os conceitos do **private data** do framework **Hyperledger Fabric**, seu funcionamento, regras de **policy**, etc. Mas para contextualizar de forma simplória, quando um ativo é definido como private, apenas o hash do seu conteúdo fica gravado no **channel**, o conteúdo do asset (suas propriedades) apenas são acessados em um conjunto limitado de organizações, sendo essas informaçãos registradas em bases de dados base de dados transientes em cada peer.


- O conteúdo do ativo apenas pode ser lido pela **org2** e **org3**
- Chave primária **secretName** de tipo **string**
- Apenas a **org2** pode criar ou alterar a propriedade **secretName** e o asset **Secret** (porque é chave)
- Propriedade **secret** de tipo **string** é obrigatória para o asset
- Tanto a **org2** quanto a **org3** pode criar ou alterar a propridade **secret**

O arquivo **collections.json** precisa ser configurado de acordo com os assets que possuem o campo **Readers**

    [
            {
            "name": "secret",
            "policy": {
            "identities": [
                {
                "role": {
                    "name": "member",
                    "mspId": "org2MSP"
                }
                },
                {
                "role": {
                    "name": "member",
                    "mspId": "org3MSP"
                }
                }
            ],
            "policy": {
                "1-of": [
                {
                    "signed-by": 0
                },
                {
                    "signed-by": 1
                }
                ]
            }
            },
            "requiredPeerCount": 0,
            "maxPeerCount": 3,
            "blockToLive": 1000000,
            "memberOnlyRead": true
        }
    ]


## Asset list definition

O cadastro dos assets que serão usados pela biblioteca **GoLedger CC-Tools** deve ser realizado no arquivo **chaincode/assetTypeList.go**

    var assetTypeList = []assets.AssetType{
        assettypes.Person,
        assettypes.Book,
        assettypes.Library,
        assettypes.Secret,
    }
