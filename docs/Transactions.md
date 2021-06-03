# Transactions

**GoLedger CC-Tools** transactions represent the **GoLang** methods that can modify the assets within the Blockchain ledger (**Hyperledger Fabric Channel**)

**Hyperledger Fabric** assets can only be created or modified by executing **chaincode** transactions inside endorsing peers.

**Goledger CC-Tools** library has a range of pre-defined transactions:

- `CreateAsset` - creation of a new asset
- `UpdateAsset` - update an existing asset
- `DeleteAsset` - removing an asset from the current ledger state
- `ReadAsset` - read the asset in its last ledger status
- `ReadAssetHistory` - history of an asset's ledger status
- `Search` - asset listing

**GoLedger CC-Tools** library enables the creation of custom transactions.

**cc-tools-demo** repository provides 3 example transactions:

- `CreateNewLibrary` - create a new asset of type **Library**
- `GetNumberOfBooksFromLibrary` - returns the number of assets **Book** inside asset **Library**
- `UpdateBookTenant` - update field **currentTenant** inside asset **Book**

The definition of assets is done in the **chaincode/txdefs** folder for **GoLedger CC-Tools** library.

The of files from the **txdefs** folder is shown below:

    chaincode/
        txdefs/                             # transations folder
            createNewLibrary.go             # library creation
            getNumberOfBooksFromLibrary.go  # returns the number of books at the library asset
            updateBookTenant.go             # changes de tenant of book
    txList.go                               # list of custom transactions

## Transaction definition

A custom transaction construction is done by creating a file inside the **chaincode/txdefs** folder

An asset has the following fields:

- `Tag`: string field to define the transaction name internally referenced by the code and by the Rest Api endpoints. It cannot have spaces or special characters. If it has the same name as a predefined transaction (**createAsset**, **updateAsset**, **deleteAsset**, **readAsset**, **readAssetHistory** or **search**), this transaction will be replaced by the one in the **txdefs** directory.
- `Label`: string field to define the label to be used by external applications. Free text.
- `Description`: asset description string field to be used by external applications. Free text.
- `Method`: Rest API method type. It can be **POST**, **GET**, **PUT** or **DELETE**.
- `Callers`: used to define which organizations can call this transaction.
- `Args`: arguments. It has its own fields.
- `Routine`: transaction source code.

## Transaction argument definition

A transaction has a set of customizable input arguments.

An argument has the following fields:

- `Tag`: string field to define the argument name internally referenced by the code and by the Rest Api endpoints. It cannot have spaces or special characters.
- `Label`: string field to define the label to be used by external applications. Free text.
- `Description`: asset description string field to be used by external applications. Free text.
- `Required`: identifies if the argument is required. Boolean field.
- `DataType`: property type. CC-Tools has the following default types: **string, number, datetime and boolean**.

## Transaction examples

**cc-tools-demo** repository has the following custom transaction configuration:

    chaincode/
        txdefs/                             # transations folder
            createNewLibrary.go             # library creation
            getNumberOfBooksFromLibrary.go  # returns the number of books at the library asset
            updateBookTenant.go             # changes de tenant of book
    txList.go                               # list of custom transactions

In addition to the files for each transaction that can be used by the **Goledger CC-Tools** library they also must be registered inside **txList.go** file

The definition of the **CreateNewLibrary** transaction is as follows:

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

According to the description above, **CreateNewLibrary** transaction has the following characteristics:

- Only **org3** can call this method
- **POST** method for Rest Api
- Argument **name** of type **string** is required.
- The transaction uses the **NewAsset** function to prepare a new asset (keys, etc) and the **PutNew** function to create the asset in the **channel**.

The definition of the **UpdateBookTenant** transaction is as follows:

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

- Any organization that starts with **"org"** followed by a number (org1, org2, org3, org4, etc) can call this method.
- **PUT** method for Rest Api
- Argument **book** of type **Book** is mandatory.
- Argument **person** of type **Person** required.
- The transaction uses the **Get** function to read the the assets **Book** and **Person** information from the ledger
- The transaction uses the **Update** function to updates the asset information **Book** inside the ledger

The definition of the **GetNumberOfBooksFromLibrary** transaction is as follows:

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

- Only **org2** can call this method.
- **GET** method for Rest Api
- Argument **library** of type **Library** is required.
- The transaction uses the **Get** function to read **Library** asset information from the ledger

## Transaction list definition

The registration of the transactions that will be used by **GoLedger CC-Tools** library must be also done inside **chaincode/txList.go** file

    var txList = []tx.Transaction{
        txdefs.GetHeader,
        txdefs.CreateNewLibrary,
        txdefs.GetNumberOfBooksFromLibrary,
        txdefs.UpdateBookTenant,
    }
