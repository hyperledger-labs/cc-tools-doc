# Datatypes

**GoLedger CC-Tools**  datatypes are a way to categorize and define the nature of a data that can be stored in an asset and manipulated in transactions. It's essential to validate that the data entered matches the expected data type.

CC-Tools provides some primitive datatypes. Here are them:

-   `string`: represents a sequence of characters.
-   `number`: represents a 64 bits floating-point.
-   `integer`: represents whole numbers The size depends on the underlying architecture and can vary between 32 bits and 64 bits
-   `boolean`: represents two possible values: true and false.
-   `datetime`: represents a date in RFC3339 format, e.g. "2019-05-06T22:12:41Z".
-   `@object`: represents a string in JSON format. It is typically used to define the structure and properties of a JSON object. It's made up of key-value pairs.

It also allows developers to inject custom primitive data types. In order to use  **GoLedger CC-Tools**  library, the datatypes definition is done in the  **chaincode/datatypes**  folder

Here follow the list of files:

```
chaincode/
    datatypes/        # custom datatype folders
        bookType.go   # definition of a book type
        cpf.go        # definition of a cpf
        datatypes.go  # list of datatypes instantiated

```

## Datatype definition

The construction and definition of a custom datatype is done by creating a file at  **chaincode/datatypes**  folder

An datatype has the following fields

-   `AcceptedFormats`: list of "core" types that can be accepted (string, number, integer, boolean, datetime).
-   `Description`: text describing the data type.
-   `DropDownValues`: set of predetermined values to be used in a dropdown menu on frontend rendering.
-   `Parse`: function called to check if the input value is valid, make necessary conversions and return a string representation of the value.

## Datatype examples

The  `cc-tools-demo`  repository has the following examples:

```
chaincode/
    datatypes/     # datatypes folders
        bookType.go   # definition of a book type
        cpf.go        # definition of a cpf
        datatypes.go  # list of datatypes instantiated

```

Besides the files of each datatype, you must register the datatypes that can be used by the  **Goledger CC-Tools**  library in the  **datatypes.go**  file

The definition of the  **bookType**  datatype is as follows:

```
type BookType float64

const (
    BookTypeHardcover BookType = iota
    BookTypePaperback
    BookTypeEbook
)

// CheckType checks if the given value is defined as valid BookType consts
func (b BookType) CheckType() errors.ICCError {
    switch b {
    case BookTypeHardcover:
        return nil
    case BookTypePaperback:
        return nil
    case BookTypeEbook:
        return nil
    default:
        return errors.NewCCError("invalid type", 400)
    }

}

var bookType = assets.DataType{
    AcceptedFormats: []string{"number"},
    DropDownValues: map[string]interface{}{
        "Hardcover": BookTypeHardcover,
        "Paperback": BookTypePaperback,
        "Ebook":     BookTypeEbook,
    },
    Description: ``,

    Parse: func(data interface{}) (string, interface{}, errors.ICCError) {
        var dataVal float64
        switch v := data.(type) {
        case float64:
            dataVal = v
        case int:
            dataVal = (float64)(v)
        case BookType:
            dataVal = (float64)(v)
        case string:
            var err error
            dataVal, err = strconv.ParseFloat(v, 64)
            if err != nil {
                return "", nil, errors.WrapErrorWithStatus(err, "asset property must be an integer, is %t", 400)
            }
        default:
            return "", nil, errors.NewCCError("asset property must be an integer, is %t", 400)
        }

        retVal := (BookType)(dataVal)
        err := retVal.CheckType()
        return fmt.Sprint(retVal), retVal, err
    },
}

```

According to the description above, the datatype  **bookType**  has the following characteristics:

-   It represents an enumerated type
-   Accepted values are the numbers 0, 1 and 2, which represent book types  **Hardcover**,  **Paperback**  and  **Ebook**, respectively
-   Parse function receives the input value, parse it to  `float64`and check number is valid in  `CheckType`  function

The definition of the  **cpf**  datatype is as follows:

```
var cpf = assets.DataType{
	AcceptedFormats: []string{"string"},
	Parse: func(data interface{}) (string, interface{}, errors.ICCError) {
		cpf, ok := data.(string)
		if !ok {
			return "", nil, errors.NewCCError("property must be a string", 400)
		}

		cpf = strings.ReplaceAll(cpf, ".", "")
		cpf = strings.ReplaceAll(cpf, "-", "")

		if len(cpf) != 11 {
			return "", nil, errors.NewCCError("CPF must have 11 digits", 400)
		}

		var vd0 int
		for i, d := range cpf {
			if i >= 9 {
				break
			}
			dnum := int(d) - '0'
			vd0 += (10 - i) * dnum
		}
		vd0 = 11 - vd0%11
		if vd0 > 9 {
			vd0 = 0
		}
		if int(cpf[9])-'0' != vd0 {
			return "", nil, errors.NewCCError("Invalid CPF", 400)
		}

		var vd1 int
		for i, d := range cpf {
			if i >= 10 {
				break
			}
			dnum := int(d) - '0'
			vd1 += (11 - i) * dnum
		}
		vd1 = 11 - vd1%11
		if vd1 > 9 {
			vd1 = 0
		}
		if int(cpf[10])-'0' != vd1 {
			return "", nil, errors.NewCCError("Invalid CPF", 400)
		}

		return cpf, cpf, nil
	},
}

```

According to the description above, the  **cpf**  datatype has the following characteristics:

-   It accepts a string as input in the CPF standard format, e.g. "861.232.710-59". Punctuation is not required.
-   Parse function validates that the provided CPF numbers adhere to the correct structure, including the required digits and verification algorithm.

## Datatype list definition

**GoLedger CC-Tools**  custom datatypes registration must be defined in the  **chaincode/datatypes/datatypes.go**  file**

```
var CustomDataTypes = map[string]assets.DataType{
	"cpf":      cpf,
	"bookType": bookType,
}
```