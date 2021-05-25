# Transactions

As transações dentro da biblioteca **GoLedger CC-Tools** representam os métodos em **GoLang** que podem modificar os ativos dentro do ledger do Blockchain (**Hyperledger Fabric Channel**)

No **Hyperledger Fabric** os assets apenas podems ser criados ou modificados através da execução de transações dos **chaincode**

A biblioteca **Goledger CC-Tools** possui um gama de transações pré-definidas:

- `CreateAsset`- criação de um novo asset
- `UpdateAsset` - atualização de um asset existente
- `DeleteAsset` - remoção de um asset do estado atual do ledger
- `ReadAsset` - leitura do asset no seu último estado no ledger
- `ReadAssetHistory` - histórico dos estados de um asset no ledger
- `Search` - listagem de assets

A biblioteca **GoLedger CC-Tools** possibilita a criação de transações customizadas.

No repositório **cc-tools-demo** existem 3 transações de exemplo:

- `CreateNewLibrary` - criar um novo asset do tipo **Library**
- `GetNumberOfBooksFromLibrary` - retorna o número de assets **Book** 
- `UpdateBookTenant` - atualizar o campo **currentTenant** no asset **Book**

Para uso da biblioteca GoLedger CC-Tools, a definição dos assets é feito na pasta **chaincode/txdefs**

A seguir está lista de arquivos da pasta **txdefs**:

    chaincode/
        txdefs/                             # transations folder
            createNewLibrary.go             # library creation
            getNumberOfBooksFromLibrary.go  # returns the number of books at the library asset
            updateBookTenant.go             # changes de tenant of book
    txList.go                               # list of custom transactions

## Transaction definition

A construção e definição de uma transação customizada se faz com a criação de um arquivo dentro da pasta **chaincode/txdefs**

Um asset possui os seguintes campos:

- `Tag`: campo string para definir o nome da transação referenciado internamente pelo código e pela Rest Api. Não pode ter espaços ou caracteres especiais. Caso tenha o mesmo nome de alguma transação pré-definida (**createAsset**, **updateAsset**, **deleteAsset**, **readAsset**, **readAssetHistory** ou **search**), essa transação será substituída pela presente no diretóro **txdefs**.
- `Label`: campo string para definir o rótulo para ser utilizado nas aplicações externas. Texto livre.
- `Description`: campo string de descrição do asset para ser utilizado nas aplicações externas. Texto livre.
- `Method`: tipo de método da Rest Api. Pode ser **POST**, **GET**.
- `Callers`: utilizado para definir quais as organizações podem chamar essa transação.
- `Args`: argumentos. Possui campos proprios.
- `Routine`: codigo-fonte da transação.

## Transaction argument definition

Uma transação possui um conjunto de arguments de entrada customizáveis.

Um argumento possui os seguintes campos:

- `Tag`: campo string para definir o nome do argumento referenciado internamente pelo código e pela Rest Api. Não pode ter espaços ou caracteres especiais. 
- `Label`: campo string para definir o rótulo para ser utilizado nas aplicações externas. Texto livre.
- `Description`: campo string de descrição do asset para ser utilizado nas aplicações externas. Texto livre.
- `Required`: identifica se o argumento é obrigatório. Campo booleano.
- `DataType`: tipo da propriedade. CC-Tools possui os seguintes tipos padrão: **string, number, datetime e boolean**.

## Transaction examples

O repositório **cc-tools-demo** traz a seguinte configuração de transações customizadas:

    chaincode/
        txdefs/                             # transations folder
            createNewLibrary.go             # library creation
            getNumberOfBooksFromLibrary.go  # returns the number of books at the library asset
            updateBookTenant.go             # changes de tenant of book
    txList.go                               # list of custom transactions

Além dos arquivos de cada transação, deve-se cadastrar as transações que podem ser utilizadas pela biblioteca **Goledger CC-Tools** no arquivo **txList.go**

A definição da transação **CreateNewLibrary** é a seguinte:

    var CreateNewLibrary = tx.Transaction{
        Tag:         "createNewLibrary",
        Label:       "Create New Library",
        Description: "Create a New Library",
        Method:      "POST",
        Callers:     []string{"$org3MSP"}, // Only org3 can call this transaction

        Args: []tx.Argument{
            {
                Tag:         "name",
                Label:       "Name",
                Description: "Name of the library",
                DataType:    "string",
                Required:    true,
            },
        },
        Routine: func(stub shim.ChaincodeStubInterface, req map[string]interface{}) ([]byte, errors.ICCError) {
            name, ok := req["name"].(string)
            if !ok {
                return nil, errors.WrapError(nil, "Parameter name must be string")
            }

            libraryMap := make(map[string]interface{})
            libraryMap["@assetType"] = "library"
            libraryMap["name"] = name

            libraryAsset, err := assets.NewAsset(libraryMap)
            if err != nil {
                return nil, errors.WrapError(err, "Failed to create a new asset")
            }

            // Save the new library on channel
            _, err = libraryAsset.PutNew(stub)
            if err != nil {
                return nil, errors.WrapError(err, "Error saving asset on blockchain")
            }

            // Marshal asset back to JSON format
            libraryJSON, nerr := json.Marshal(libraryAsset)
            if nerr != nil {
                return nil, errors.WrapError(nil, "failed to encode asset to JSON format")
            }

            return libraryJSON, nil
        },
    }

De acordo com a descrição, a transação **CreateNewLibrary** tem as seguintes caracteríticas:

- Apenas a **org3** pode chamar esse método
- Método **POST** para a Rest Api
- Argumento **name** de tipo **string** obrigatório.
- A transação utiliza a função **NewAsset** para prepara um novo asset (chaves, etc) e a função **PutNew** para criar o asset no **channel**.

A definição da transação **UpdateBookTenant** é a seguinte:

    var UpdateBookTenant = tx.Transaction{
        Tag:         "updateBookTenant",
        Label:       "Update Book Tenant",
        Description: "Change the tenant of a book",
        Method:      "PUT",
        Callers:     []string{`$org\dMSP`}, // Any orgs can call this transaction

        Args: []tx.Argument{
            {
                Tag:         "book",
                Label:       "Book",
                Description: "Book",
                DataType:    "book",
                Required:    true,
            },
            {
                Tag:         "tenant",
                Label:       "tenant",
                Description: "New tenant of the book",
                DataType:    "person",
            },
        },
        Routine: func(stub shim.ChaincodeStubInterface, req map[string]interface{}) ([]byte, errors.ICCError) {
            bookKey, ok := req["book"].(assets.Key)
            if !ok {
                return nil, errors.WrapError(nil, "Parameter book must be an asset")
            }
            tenantKey, ok := req["tenant"].(assets.Key)
            if !ok {
                return nil, errors.WrapError(nil, "Parameter tenant must be an asset")
            }

            // Returns Book from channel
            bookAsset, err := bookKey.Get(stub)
            if err != nil {
                return nil, errors.WrapError(err, "failed to get asset from the ledger")
            }
            bookMap := (map[string]interface{})(*bookAsset)
            if bookMap["@assetType"].(string) != "book" {
                return nil, errors.WrapError(err, "failed to get solicitacao from the ledger")
            }

            // Returns person from channel
            tenantAsset, err := tenantKey.Get(stub)
            if err != nil {
                return nil, errors.WrapError(err, "failed to get asset from the ledger")
            }
            tenantMap := (map[string]interface{})(*tenantAsset)
            if tenantMap["@assetType"].(string) != "person" {
                return nil, errors.WrapError(err, "failed to get solicitacao from the ledger")
            }

            // Update data
            bookMap["tenant"] = tenantMap

            bookMap, nerr := bookAsset.Update(stub, bookMap)
            if nerr != nil {
                return nil, errors.WrapError(err, "failed to update asset")
            }

            // Marshal asset back to JSON format
            bookJSON, nerr := json.Marshal(bookAsset)
            if nerr != nil {
                return nil, errors.WrapError(err, "failed to marshal response")
            }

            return bookJSON, nil
        },
    }

- Qualquer organização que comece com **"org"** seguida de um número (org1, org2, org3, org4, etc) pode chamar esse método.
- Método **PUT** para a Rest Api
- Argumento **book** de tipo **Book** obrigatório.
- Argumento **person** de tipo **Person** obrigatório.
- A transação utiliza a função **Get** para ler as informações dos assets **Book** e **Person** do ledger
- A transação utiliza a função **Update** atualiza as informação do asset **Book** no ledger

A definição da transação **GetNumberOfBooksFromLibrary** é a seguinte:

    var GetNumberOfBooksFromLibrary = tx.Transaction{
        Tag:         "getNumberOfBooksFromLibrary",
        Label:       "Get Number Of Books From Library",
        Description: "Return the number of books of a library",
        Method:      "GET",
        Callers:     []string{"$org2MSP"}, // Only org2 can call this transactions

        Args: []tx.Argument{
            {
                Tag:         "library",
                Label:       "Library",
                Description: "Library",
                DataType:    "library",
                Required:    true,
            },
        },
        Routine: func(stub shim.ChaincodeStubInterface, req map[string]interface{}) ([]byte, errors.ICCError) {
            libraryKey, ok := req["library"].(assets.Key)
            if !ok {
                return nil, errors.WrapError(nil, "Parameter library must be an asset")
            }

            // Returns Library from channel
            libraryAsset, err := libraryKey.Get(stub)
            if err != nil {
                return nil, errors.WrapError(err, "failed to get asset from the ledger")
            }
            libraryMap := (map[string]interface{})(*libraryAsset)
            if libraryMap["@assetType"].(string) != "library" {
                return nil, errors.WrapError(err, "failed to get library from the ledger")
            }

            numberOfBooks := 0
            books, ok := libraryMap["books"].([]interface{})
            if ok {
                numberOfBooks = len(books)
            }

            var returnMap map[string]interface{}
            returnMap["numberOfBooks"] = numberOfBooks

            // Marshal asset back to JSON format
            returnJSON, nerr := json.Marshal(returnMap)
            if nerr != nil {
                return nil, errors.WrapError(err, "failed to marshal response")
            }

            return returnJSON, nil
        },
    }

- Apenas a **org2** pode chamar esse método.
- Método **GET** para a Rest Api
- Argumento **library** de tipo **Library** obrigatório.
- A transação utiliza a função **Get** para ler as informações do asset **Library** do ledger

## Transaction list definition

O cadastro das transações que serão usados pela biblioteca **GoLedger CC-Tools** deve ser realizado no arquivo **chaincode/txList.go**

    var txList = []tx.Transaction{
        txdefs.GetHeader,
        txdefs.CreateNewLibrary,
        txdefs.GetNumberOfBooksFromLibrary,
        txdefs.UpdateBookTenant,
    }
