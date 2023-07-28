# Assets

Blockchain assets represent the information that will be used by the Blockchain ledger (**Hyperledger Fabric Channel**)

Roughly speaking, an asset can be compared to a table in a relational database.

And just like in databases, assets are formed by properties, some of which can be part of a set of keys for that asset. Each property can have a specific data type, such as string, number, boolean, etc.

An asset can also be a **Hyperledger Fabric Private Data**, in which the information content is recorded in a transient database outside the ledger and only a subset of organizations receive this data, configuring readability.

Assets can also be created dynamically during the chaincode execution, using CC-Tool's Dynamic Asset Types funcionality, in which asset types are defined using invoke calls to the running chaincode. 

For the cc-tools-demo repository, there are 4 assets (one of which is a private data)

- Person
- Book
- Library
- Secret (private data)

In order to use **GoLedger CC-Tools** library, the assets definition is done in the **chaincode/assettypes** folder

Here follow the list of files:

    chaincode/
        assettypes/     		  # asset folders
            person.go   		  # definition of a person
            book.go     		  # definition of a book
            library.go  		  # definition of a library
            secret.go             # definitions of secret (private data)
			customAssets.go       # list of assets inserted via GoFabric's Template mode
			dynamicAssetTypes.go  # configuration file for Dynamic Asset Types
    assetTypeList.go   			  # list of assets instantiated

## Asset definition

The construction and definition of an asset is done by creating a file at **chaincode/assettypes** folder

An asset has the following fields

- `Tag`: string field used to define the name of the asset referenced internally by the code and by the Rest API endpoints. No space or special characters allowed.
- `Label`: string field to define the label to be used by external applications. Free text.
- `Description`: asset description string field to be used by external applications. Free text.
- `Props`: asset properties. It has its own fields (description next)
- `Readers`: used to define private data organizations (needs additional configuration in **collections2.json** file)
- `Validate`: asset validation function. Allows for a custom code to validate the asset as a whole.
- `Dynamic`: boolean field to indicate that the asset type can be modified dynamically. For coded assets it is recommended to leave it as *false* and use the upgrade chaincode funcionality to update the asset definition.


## Property definition

An asset has a set of properties (Props) to structure the asset.

A property has the following fields:

- `Tag`: string field used to define the name of the property referenced internally by the code and by the Rest API endpoints. No space or special characters allowed.
- `Label`: string field to define the label to be used by external applications. Free text.
- `Description`: property description string field to be used by external applications. Free text.
- `IsKey`: identifies if the property is part of the asset's keys. Boolean field.
- `Required`: identifies if the property is mandatory. Boolean field.
- `ReadOnly`: identifies if the property can no longer be modified once created. Boolean field.
- `DefaultValue`: property default value.
- `Writers`: define the organizations that can create or change this property. If the property is key (isKey field: true) then the entire asset can only be created by the organization. List of strings.
- `DataType`: property type. CC-Tools has the following default types: **string, number, datetime and boolean**. Custom types can be defined in the **chaincode/datatypes** folder. Arrays and references to other asset types are also possible.
- `Validate`: property validation function. It is suggested only for simple validations, more complex functions should use custom datatypes.

## Asset examples

The `cc-tools-demo` repository has the following examples:

    chaincode/
        assettypes/     		  # asset foldgers
            person.go   		  # definition of a person
            book.go     		  # definition of a book
            library.go  		  # definition of a library
            secret.go   		  # definitions of secret (private data)
			customAssets.go       # list of assets inserted via GoFabric's Template mode
			dynamicAssetTypes.go  # configuration file for Dynamic Asset Types
    assetTypeList.go    		  # list of assets instantiated

Besides the files of each asset, you must register the assets that can be used by the **Goledger CC-Tools** library in the **assetTypeList.go** file

The definition of the **Person** asset is as follows:

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
			    Writers:  []string{`org1MSP`, "orgMSP"}, // This means only org1 and org can create the asset (others can edit)
		    },
		    {
    			// Mandatory property
			    Required: true,
			    Tag:      "name",
			    Label:    "Name of the person",
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
			    Tag:          "height",
			    Label:        "Person's height",
			    DefaultValue: 0,
			    DataType:     "number",
		    },
	    },
    }

According to the description above, the asset **Person** has the following characteristics:

- Primary key **cpf** (custom datatype)
- Only **org1** and **org** can create or change the **cpf** property and the **Person** asset (because this is a key)
- Property **name** of type **string** is mandatory, and has automatic validation to prevent empty strings ("")
- **dateofBirth** property of type **datetime** optional.
- **height** property of type **number**, with default value 0

The definition of the **Book** asset is as follows:

    var Book = assets.AssetType{
    	Tag:         "book",
    	Label:       "Book",
    	Description: "Book",

    	Props: []assets.AssetProp{
		    {
			    // Composite Key
			    Required: true,
			    IsKey:    true,
			    Tag:      "title",
			    Label:    "Book Title",
			    DataType: "string",
			    Writers:  []string{`org2MSP`, "orgMSP"}, // This means only org2 and org can create the asset (others can edit)
		    },
		    {
    			// Composite Key
			    Required: true,
			    IsKey:    true,
			    Tag:      "author",
			    Label:    "Book Author",
			    DataType: "string",
			    Writers:  []string{`org2MSP`, "orgMSP"}, // This means only org2 and org can create the asset (others can edit)
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
			{
				// Custom data type
				Tag:      "bookType",
				Label:    "Book Type",
				DataType: "bookType",
			},
	    },
    }

According to the description above, the **Book** asset has the following characteristics:

- Composite key **title** and **author**, both with type **string**
- Only **org2** and **org** can create or change the **title** and **author** properties and the **Book** asset (because they are keys)
- **currentTenant** property of type **->person** which represents the reference to a **Person** asset
- **genres** property of type **[]string** which represents an array of strings
- **published** property of type **datetime**
- **bookType** property is of custom datatype **bookType**, defined in **chaincode/datatypes** folder

The definition of the **Library** asset is as follows:

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
			    Writers:  []string{`org3MSP`, "orgMSP"}, // This means only org3 and org can create the asset (others can edit)
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
According to the description above, the **Library** asset has the following characteristics:

- Primary key **name** of type **string**
- Only **org3** and **org** can create or change the **name** property and the **Library** asset (because it's key)
- **books** property of type **[]->book** which represents an array of references to the **Book** asset
- **entranceCode** property of type **->secret** which represents the reference to a private data type

The definition of the **Secret** asset is as follows:

    var Secret = assets.AssetType{
	    Tag:         "secret",
    	Label:       "Secret",
	    Description: "Secret between Org2 and Org3",

    	Readers: []string{"org2MSP", "org3MSP", , "orgMSP"},
	    Props: []assets.AssetProp{
    		{
	    		// Primary Key
		    	IsKey:    true,
			    Tag:      "secretName",
			    Label:    "Secret Name",
    			DataType: "string",
			    Writers:  []string{`org2MSP`, , "orgMSP"}, // This means only org2 and org can create the asset (org3 can edit)
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

According to the description above, **Secret** asset has privacy features (**Hyperledger Fabric Private Data**). For this asset to work correctly, the **collections2.json** file must be configured

We will not describe  **Hyperledger Fabric Private Data** concepts, its operation, **policy** rules, etc. But to put it simply, when an asset is defined as private, only its content hash is recorded in **channel**, the asset's content (its properties) are only accessed by a limited set of organizations, the content recorded in transient databases in each peer.

- Asset content can only be read by **org**, **org2** and **org3**
- Primary key **secretName** of type **string**
- Only **org2** and **org** can create or change the **secretName** property and the **Secret** asset (because it is key)
- **secret** property of type **string** is mandatory for the asset
- Both **org**, **org2** and **org3** can create or change the **secret** property

The **collections2.json** file needs to be configured according to the assets that have the **Readers** field

    [
        {
            "name": "secret",
            "requiredPeerCount": 0,
            "maxPeerCount": 3,
            "blockToLive": 1000000,
            "memberOnlyRead": true,
			"policy": "OR('org2MSP.member', 'org3MSP.member')"
        }
    ]

For development networks started with only 1 organization **org**, the **collections2-org.json** is used instead.

    [
        {
            "name": "secret",
            "requiredPeerCount": 0,
            "maxPeerCount": 3,
            "blockToLive": 1000000,
            "memberOnlyRead": true,
			"policy": "OR('orgMSP.member')"
        }
    ]

## Asset list definition

**GoLedger CC-Tools** assets registration must be defined in the **chaincode/assetTypeList.go** file**

    var assetTypeList = []assets.AssetType{
        assettypes.Person,
        assettypes.Book,
        assettypes.Library,
        assettypes.Secret,
    }

## Dynamic Asset Types
