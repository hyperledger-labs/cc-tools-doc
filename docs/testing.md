# Testing

**GoLedger CC-Tools** has different ways for testing the source code in development mode.

For **GoLang** syntax checking, execute the following command:

```sh
cd chaincode \
go vet
```

After the succesfull code instantiation or update, you can check the logs directly inside the **chaincode** execution **peers** containers. These containers can be identified by starting with `dev`.

```sh
docker logs dev-peer0.org1.example.com-cc-tools-demo-0.1
```

## Using cc-webclient

To perform the tests and integrations satisfactorily, it is suggested the usage of **cc-webclient** tool. After finishing the steps, it is suggested to execute the following commands to create 3 applications in order to connect with **org1, org2 and org3**.

```sh
./run-cc-web.sh 8080 & \
./run-cc-web.sh 8090 & \
./run-cc-web.sh 8100 &
```

### Configuring rest-server and cc-webclient

After executing **cc-webclient** containers, the applications can be accessed directly through the ports defined in the script. E.g.: 8080, 8090, 8100

After accessing **cc-webclient** through the browser, the configuration of rest server address can be done by clicking on the tool icon.

![Config](img/header-cc-webclient.png)

Accesses are can be made with the following settings:

- `org1`: **http://localhost:80**
- `org2`: **http://localhost:980**
- `org3`: **http://localhost:1080**

![Config](img/config-cc-webclient.png)

**cc-webclient** application has a sidebar that shows the available **assets** as well as the **transactions** registered in the chaincode.

![Bar](img/bar-cc-webclient.png)

### Endpoint usage

The rest-server endpoint usage is shown using the **CURL** buttons.

![CURL](img/curl-cc-webclient.png)

For each screen, you can check endpoint usage pressing the `curl` button.

For example, on the asset creation screen:

![CURL](img/curl-create-cc-webclient.png)

### List, create, edit, delete or history an asset

To **list** each asset, just select an asset in the sidebar.

![List Asset](img/list-cc-webclient.png)

Selecting the `CREATE` button at the asset list window, an asset **creation** screen will appear.

For example, for the **Person** asset

![Create Asset](img/create-asset-cc-webclient.png)

The **edit** screen is accessed by selecting the **edit icon** in an asset's list window.

The **removal** of an asset is requested by selecting the **delete icon** in the asset's list window.

The **history** of an asset (all changes recorded in the ledger) can be viewed by selecting the **history icon** in an asset's window.

### Executing an transaction

The **transaction** execution can be performed by selecting the transaction in the sidebar.

For example, for transaction **UpdateBookTenant**

![Transaction](img/transaction-cc-webclient.png)

## Using Godog
`Godog`  is the official Cucumber BDD framework for Golang. It follows the principles of BDD, which emphasizes defining the desired behavior of the software through user-focused scenarios written in natural language. These scenarios are written in a Given-When-Then format, making it easier for non-technical team members to understand and contribute to the testing process.

For the cc-tools-demo repository, there are godog tests for every transaction. Each transaction is represented as a feature and each feature has scenarios to be tested.

Features are defined in  `chaincode/tests/features`  folder.

```
chaincode/
  tests/
    features/
      createNewLibrary.feature              # definition of Create New Library feature
      getBooksByAuthor.feature              # definition of Get Books By Author feature
      getNumberOfBooksFromLibrary.feature   # definition of Get Number Of Books From Library feature
      updateBookTenant.feature              # definition of Create New Library feature
    request_test.go                         # implementation of steps for scenarios

```

### Feature definition

In the feature file, you start by defining the  **feature**  using the  `Feature`  keyword followed by a brief description. Each  **scenario**  within the feature is defined using the  `Scenario`  keyword, followed by a scenario title or description. Scenarios represent specific use cases or situations that you want to test. Inside each scenario, you define the  **steps**  of the test using  `Given`,  `When`, and  `Then`  keywords. These steps describe the preconditions (Given), actions (When), and expected outcomes (Then) of the scenario. Optionally, you can also use  `And`  and  `But`  to further clarify or add additional steps.

See more on  [Godog/Cucumber](https://github.com/cucumber/godog).

### Feature examples

The definition of **Create New Library** feature is as follows:

```gherkin
Feature: Create New Library
  In order to create a new library
  As an API client
  I want to make a request with the name of the desired library

  Scenario: Create a new library
      Given there is a running "" test network from scratch
      When I make a "POST" request to "/api/invoke/createNewLibrary" on port 880 with:
          """
          {
              "name": "Elizabeth's Library"
          }
          """
      Then the response code should be 200
      And the response should have:
          """
          {
              "@key":         "library:9cf6726a-a327-568a-baf1-5881393073bf",
              "@lastTouchBy": "orgMSP",
              "@lastTx":      "createNewLibrary",
              "@assetType":   "library",
              "name":         "Elizabeth's Library"
          }
          """

  Scenario: Try to create a new library with a name that already exists
      Given there is a running "" test network
      Given there is a library with name "John's Library"
      When I make a "POST" request to "/api/invoke/createNewLibrary" on port 880 with:
          """
          {
              "name": "John's Library"
          }
          """
      Then the response code should be 409

```

This feature tests the ability to create a new library using the  `createNewLibrary`  tx through the API. It consists of two scenarios: one is a successful case, and the other tests when attempting to create a library with a name that already exists.

-   **Scenario 1:**  This scenario represents the successful creation of a new library. It verifies that the API can correctly handle the request with a new library name and returns the expected response.
-   **Scenario 2:**  This scenario tests the case when attempting to create a library with a name that already exists in the system. It checks whether the API responds with the appropriate status code (409) to indicate a conflict.

The definition of  **Get Books By Author**  feature is as follows:

```gherkin
Feature: Get Books By Author
  In order to get all the books by an author
  As an API client
  I want to make a request to the getBooksByAuthor transaction
  And receive the appropriate books

  Scenario: Request an author with multiple books
      Given there is a running "" test network
      And there are 3 books with prefix "book" by author "Jack"
      When I make a "GET" request to "/api/query/getBooksByAuthor" on port 880 with:
          """
          {
              "authorName": "Jack"
          }
          """
      Then the response code should be 200
      And the "result" field should have size 3

  Scenario: Request an author with no books
      Given there is a running "" test network
      When I make a "GET" request to "/api/query/getBooksByAuthor" on port 880 with:
          """
          {
              "authorName": "Mary"
          }
          """
      Then the response code should be 200
      And the "result" field should have size 0

  Scenario: Request an author with 2 books while there are other authors with more books
      Given there is a running "" test network
      Given there are 1 books with prefix "fantasy" by author "Missy"
      Given there are 2 books with prefix "cook" by author "John"
      When I make a "GET" request to "/api/query/getBooksByAuthor" on port 880 with:
          """
          {
              "authorName": "John"
          }
          """
      Then the response code should be 200
      And the "result" field should have size 2

```

This feature tests the ability to retrieve books written by a specific author using the  `getBooksByAuthor`  transaction through the API. It consists of three scenarios that cover different cases based on the number of books by the requested author.

-   **Scenario 1:**  This scenario verifies that the API can successfully handle a request for an author who has multiple books in the system. It checks whether the API returns the correct response code (200) and that the "result" field in the response contains all the books by the author with a size of 3.
-   **Scenario 2:**  This scenario tests the case when requesting an author who has no books in the system. It checks whether the API returns the appropriate response code (200) and that the "result" field in the response is empty with a size of 0.
-   **Scenario 3:**  This scenario validates the API's response when requesting an author with two books, while other authors have more books in the system. It verifies whether the API returns the correct response code (200) and that the "result" field contains the two books by the requested author with a size of 2.

The definition of  **Get Number Of Books From Library**  feature is as follows:

```gherkin
Feature: Get Number Of Books From Library
  In order to create the number of books from library
  As an API client
  I want to make a request

  Scenario: Query Get Number Of Books From Library that Exists
      Given there is a running "" test network
      And I make a "POST" request to "/api/invoke/createAsset" on port 880 with:
          """
          {
              "asset": [
                  {
                      "@assetType": "book",
                      "title":      "Meu Nome é Maria",
                      "author":     "Maria Viana"
                  }
              ]
          }
          """
      And I make a "POST" request to "/api/invoke/createAsset" on port 880 with:
          """
          {
              "asset": [{
                  "@assetType": "library",
                  "name": "Maria's Library",
                  "books": [
                      {
                          "@assetType": "book",
                          "@key": "book:a36a2920-c405-51c3-b584-dcd758338cb5"
                      }
                  ]
              }]
        }
          """
      When I make a "GET" request to "/api/query/getNumberOfBooksFromLibrary" on port 880 with:
          """
          {
              "library": {
                  "@key": "library:3cab201f-9e2b-579d-b7b2-72297ed17f49",
                  "@assetType": "library"
          }
          }
          """
      Then the response code should be 200
      And the response should have:
          """
          {
              "numberOfBooks": 1.0
          }
          """

  Scenario: Query Get Number Of Books From Library that Does Not Exists
      Given there is a running "" test network
      When I make a "GET" request to "/api/query/getNumberOfBooksFromLibrary" on port 880 with:
          """
          {
              "library": {
                  "@key": "library:5c5b201f-9e4c-579d-b7b2-72297ed17f78",
                  "@assetType": "library"
          }
          }
          """
      Then the response code should be 400

```

This feature tests the ability to retrieve the number of books from a library through the API. It consists of two scenarios: one scenario deals with querying a library that exists and has books, while the other scenario tests querying a library that does not exist.

-   **Scenario 1:**  This scenario verifies that the API can successfully retrieve the number of books from a library that exists in the system. It first creates a new book and a library with the book associated. Then it queries the number of books from the library and checks whether the API returns the correct response code (200) and whether the response contains the expected number of books (1).
-   **Scenario 2:**  This scenario tests the case when attempting to query the number of books from a library that does not exist in the system. It checks whether the API returns the appropriate response code (400) to indicate a bad request due to the non-existent library.

The definition of  **Update Book Tentant**  feature is as follows:

```gherkin
Feature: Update Book Tentant
  In order to update book tentant
  As an API client
  I want to make a request

  Scenario: Update Book With A Existing Tentant 
      # The first 3 statements will be used by all scenarios on this feature
      Given there is a running "" test network
      And I make a "POST" request to "/api/invoke/createAsset" on port 880 with:
          """
          {
              "asset": [{
                      "@assetType": "book",
                      "title":      "Meu Nome é Maria",
                      "author":     "Maria Viana"
                  }]
          }
          """
      And I make a "POST" request to "/api/invoke/createAsset" on port 880 with:
          """
         {
              "asset": [{
                  "@assetType": "person",
                  "name": "Maria",
                  "id": "31820792048"
              }]
        }
          """
      When I make a "PUT" request to "/api/invoke/updateBookTenant" on port 880 with:
          """
          {
              "book": {
                  "@assetType": "book",
                  "@key": "book:a36a2920-c405-51c3-b584-dcd758338cb5"
          },
              "tenant": {
                  "@assetType": "person",
                  "@key": "person:47061146-c642-51a1-844a-bf0b17cb5e19"
              }
          }
          """
      Then the response code should be 200
      And the response should have:
          """
          {
              "@key": "book:a36a2920-c405-51c3-b584-dcd758338cb5",
              "@lastTouchBy": "orgMSP",
              "@lastTx": "updateBookTenant",
              "currentTenant": {
            "@assetType": "person",
            "@key": "person:47061146-c642-51a1-844a-bf0b17cb5e19"
          }
          }
          """

  Scenario: Update Book With A Not Existing Tentant
      Given there is a running "" test network
      When I make a "PUT" request to "/api/invoke/updateBookTenant" on port 880 with:
          """
          {
              "book": {
                  "@assetType": "book",
                  "@key": "book:a36a2920-c405-51c3-b584-dcd758338cb5"
          },
              "tenant": {
                  "@assetType": "person",
                  "@key": "person:56891146-c6866-51a1-844a-bf0b17cb5e19"
              }
          }
          """
      Then the response code should be 404

```

This feature tests the ability to update the tenant of a book using the API. It consists of two scenarios: one scenario deals with updating a book with an existing tenant, and the other scenario tests updating a book with a non-existent tenant.

-   **Scenario 1:**  This scenario verifies that the API can successfully update the tenant of a book with an existing tenant. It creates a new book and a new person (tenant), associates the person with the book, and then makes a PUT request to update the book's tenant. The scenario checks whether the API returns the correct response code (200) and whether the book's tenant is correctly updated in the response.
-   **Scenario 2:**  This scenario tests the case when attempting to update the tenant of a book with a tenant that does not exist in the system. It makes a PUT request to update the book's tenant with the non-existing tenant. The scenario checks whether the API returns the appropriate response code (404) to indicate that the tenant does not exist in the system.

### Implementation of steps

Each step is implemented in  `chaincode/tests/request_test.go`  file.

For  `Given there is a running "" test network from scratch`  there is the following implementation:

```golang
func thereIsARunningTestNetworkFromScratch(arg1 string) error {
	// Start test network with 1 org only
	cmd := exec.Command("../../startDev.sh", "-n", "1")

	_, err := cmd.Output()

	if err != nil {
		fmt.Println(err.Error())
		return err
	}

	// Wait for ccapi
	err = waitForNetwork("880")
	if err != nil {
		fmt.Println(err.Error())
		return err
	}

	return nil
}  

```

This function starts a test network from scratch with a single organization. It executes a shell command  `../../startDev.sh -n 1`  to start the test network. After starting the network, it waits for the ccapi to be available on port 880 using the  `waitForNetwork`  function.

For  `When I make a "[method]" request to "[endpoint]" on port [port] with "[body]"`  there is the following implementation:

```golang
func iMakeARequestToOnPortWith(ctx context.Context, method, endpoint string, port int, reqBody *godog.DocString) (context.Context, error) {
	var res *http.Response
	var req *http.Request
	var err error

	// Initialize http client
	client := &http.Client{}

	// Create request
	if method == "GET" {
		b64str := b64.StdEncoding.EncodeToString([]byte(reqBody.Content))
		reqParam := "?@request=" + b64str
		req, err = http.NewRequest("GET", "http://localhost:"+strconv.Itoa(port)+endpoint+reqParam, nil)
		if err != nil {
			return ctx, err
		}
	} else {
		dataAsBytes := bytes.NewBuffer([]byte(reqBody.Content))

		req, err = http.NewRequest(method, "http://localhost:"+strconv.Itoa(port)+endpoint, dataAsBytes)
		if err != nil {
			return ctx, err
		}
	}

	// Set header and make request
	req.Header.Set("Content-Type", "application/json")
	res, err = client.Do(req)
	if err != nil {
		return ctx, err
	}

	// Get status code and response body
	statusCode := res.StatusCode
	resBody, err := ioutil.ReadAll(res.Body)

	if err != nil {
		return ctx, err
	}
	res.Body.Close()

	// Append status code and response body to context
	bodyCtx := context.WithValue(ctx, bodyCtxKey{}, resBody)
	statusCtx := context.WithValue(bodyCtx, statusCtxKey{}, statusCode)
	return statusCtx, nil
}
```

This function makes an HTTP request to a specified endpoint on a given port. The function supports both  `GET`  and  `POST`  methods. If the method is  `GET`, the request body is base64 encoded and added as a query parameter. If the method is  `POST`, the request body is included in the request as JSON data.

For  `Then the response code should be [code]`  there is the following implementation:

```golang
func theResponseCodeShouldBe(ctx context.Context, expectedCode int) (context.Context, error) {
	// Get Status Code from context
	statusCode, ok := ctx.Value(statusCtxKey{}).(int)
	if !ok {
		return ctx, errors.New("context unaveilable while retrieving status")
	}

	if statusCode != expectedCode {
		// Get Response body from context
		resBody, ok := ctx.Value(bodyCtxKey{}).([]byte)
		if !ok {
			return ctx, errors.New("unaveilable context while retrieving body")
		}

		// Test Failed
		return ctx, fmt.Errorf("received wrong status response. Got %d Expected: %d\nResponse body: %s", statusCode, expectedCode, string(resBody))
	}

	return ctx, nil
}

```

This function checks whether the received HTTP response status code matches the expected code.

For  `And the response should have`  there is the following implementation:

```golang
func theResponseShouldHave(ctx context.Context, body *godog.DocString) error {
	// Get 'ResponseBody' from context
	respBody, ok := ctx.Value(bodyCtxKey{}).([]byte)
	if !ok {
		return errors.New("unavailable context")
	}

	var expected map[string]interface{}
	var received map[string]interface{}

	if err := json.Unmarshal([]byte(body.Content), &expected); err != nil {
		return err
	}
	if err := json.Unmarshal(respBody, &received); err != nil {
		return err
	}

	for key, value := range expected {
		if !reflect.DeepEqual(value, received[key]) {
			var expectedBytes []byte
			var receivedBytes []byte
			var err error
			if expectedBytes, err = json.MarshalIndent(value, "", "  "); err != nil {
				return err
			}
			if receivedBytes, err = json.MarshalIndent(received[key], "", "  "); err != nil {
				return err
			}

			return fmt.Errorf("Expected %s to be equal %s, but received %s", key, expectedBytes, receivedBytes)
		}
	}

	return nil
}
```

This function checks if the received HTTP response body matches the expected response body defined in the Godog scenario step. It first retrieves the actual response body and the expected response body from the context. It unmarshals both JSON strings into corresponding map-like structures to facilitate comparison. The function performs a deep comparison between the value of each key in the expected and received response bodies using  `reflect.DeepEqual`  function.

For  `And the "[field]" field should have size [size]`  there is the following implementation:

```golang
func theFieldShouldHaveSize(ctx context.Context, field string, expectedSize int) (context.Context, error) {
	// Get 'ResponseBody' from context
	respBody, ok := ctx.Value(bodyCtxKey{}).([]byte)
	if !ok {
		return ctx, errors.New("unavailable context")
	}

	var bodyMap map[string]interface{}

	if err := json.Unmarshal(respBody, &bodyMap); err != nil {
		return ctx, err
	}

	resultField, ok := bodyMap[field]
	if !ok {
		return ctx, errors.New("unavailable filed on body response")
	}

	fieldLen := len(resultField.([]interface{}))
	if fieldLen != expectedSize {
		// Test Failed
		return ctx, fmt.Errorf("received wrong filed size on response body. Got %d Expected: %d\nResponse body: %s", fieldLen, expectedSize, string(respBody))
	}

	return ctx, nil
}

```

This function is used to check if a specific field in the received JSON response body has the expected size (number of elements). It is typically used in Godog scenarios to verify the size of an array or a list-like field in the response.

For  `And there are [number] books with prefix "[prefix]" by author "[name]"`  there is the following implementation:

```golang
func thereAreBooksWithPrefixByAuthor(ctx context.Context, nBooks int, prefix string, author string) (context.Context, error) {
	var res *http.Response
	var err error

	for i := 1; i <= nBooks; i++ {
		// Verify if book already exists
		requestJSON := map[string]interface{}{
			"query": map[string]interface{}{
				"selector": map[string]interface{}{
					"author":     author,
					"title":      prefix + strconv.Itoa(i),
					"@assetType": "book",
				},
			},
			"resolve": true,
		}
		jsonStr, e := json.Marshal(requestJSON)
		if e != nil {
			return ctx, err
		}
		dataAsBytes := bytes.NewBuffer([]byte(jsonStr))

		if res, err = http.Post("http://localhost/api/query/search", "application/json", dataAsBytes); err != nil {
			return ctx, err
		}
		resBody, err := ioutil.ReadAll(res.Body)
		if err != nil {
			return ctx, err
		}
		res.Body.Close()

		var received map[string]interface{}
		if err = json.Unmarshal(resBody, &received); err != nil {
			return ctx, err
		}

		// Create book if it doesnt exists
		if len(received["result"].([]interface{})) == 0 {
			requestJSON := map[string]interface{}{
				"asset": []interface{}{
					map[string]interface{}{
						"author":     author,
						"title":      prefix + strconv.Itoa(i),
						"@assetType": "book",
					},
				},
			}
			jsonStr, e := json.Marshal(requestJSON)
			if e != nil {
				return ctx, err
			}
			dataAsBytes := bytes.NewBuffer([]byte(jsonStr))

			if res, err = http.Post("http://localhost:880/api/invoke/createAsset", "application/json", dataAsBytes); err != nil {
				return ctx, err
			}

			if res.StatusCode != 200 {
				return ctx, errors.New("Failed to create book asset")
			}
		}
	}

	return ctx, nil
}

```

This function is used to ensure that there are a specific number of books with a given prefix and author in the system. It is typically used in Godog scenarios to set up the required test data before running tests that involve books and authors.

For  `Given there is a library with name "[name]"`  there is the following implementation:

```golang
func thereIsALibraryWithName(ctx context.Context, name string) (context.Context, error) {
	var res *http.Response
	var err error

	// Verify if library already exists
	requestJSON := map[string]interface{}{
		"query": map[string]interface{}{
			"selector": map[string]interface{}{
				"name":       name,
				"@assetType": "library",
			},
		},
		"resolve": true,
	}
	jsonStr, e := json.Marshal(requestJSON)
	if e != nil {
		return ctx, err
	}
	dataAsBytes := bytes.NewBuffer([]byte(jsonStr))

	if res, err = http.Post("http://localhost/api/query/search", "application/json", dataAsBytes); err != nil {
		return ctx, err
	}
	resBody, err := ioutil.ReadAll(res.Body)
	if err != nil {
		return ctx, err
	}
	res.Body.Close()

	var received map[string]interface{}
	if err = json.Unmarshal(resBody, &received); err != nil {
		return ctx, err
	}

	// Create library if it doesnt exists
	if len(received["result"].([]interface{})) == 0 {
		requestJSON = map[string]interface{}{
			"asset": []interface{}{
				map[string]interface{}{
					"name":       name,
					"@assetType": "library",
				},
			},
		}
		jsonStr, e = json.Marshal(requestJSON)
		if e != nil {
			return ctx, err
		}
		dataAsBytes = bytes.NewBuffer([]byte(jsonStr))

		if res, err = http.Post("http://localhost:880/api/invoke/createAsset", "application/json", dataAsBytes); err != nil {
			return ctx, err
		}

		if res.StatusCode != 200 {
			return ctx, errors.New("Failed to create library asset")
		}
	}

	return ctx, nil
}
```

This function is used to ensure that a library with a specific name exists in the system. It is typically used in Godog scenarios to set up the required test data before running tests that involve libraries. If the library does not exist, it creates a new library asset.

It's required to initialize the scenario with the implementation of each step. The function below describes how to do it:

```golang
func InitializeScenario(ctx *godog.ScenarioContext) {
	ctx.Step(`^I make a "([^"]*)" request to "([^"]*)" on port (\d+) with:$`, iMakeARequestToOnPortWith)
	ctx.Step(`^the response code should be (\d+)$`, theResponseCodeShouldBe)
	ctx.Step(`^the response should have:$`, theResponseShouldHave)
	ctx.Step(`^there is a running "([^"]*)" test network$`, thereIsARunningTestNetwork)
	ctx.Step(`^there is a running "([^"]*)" test network from scratch$`, thereIsARunningTestNetworkFromScratch)
	ctx.Step(`^there are (\d+) books with prefix "([^"]*)" by author "([^"]*)"$`, thereAreBooksWithPrefixByAuthor)
	ctx.Step(`^the "([^"]*)" field should have size (\d+)$`, theFieldShouldHaveSize)
	ctx.Step(`^there is a library with name "([^"]*)"$`, thereIsALibraryWithName)
}
```

### Run Godog tests

Begin by installing godog using the following command:

```sh
$ go install github.com/cucumber/godog/cmd/godog@latest
```

Once godog is successfully installed, utilize the provided script to run tests effortlessly:

```sh
$ ./godog.sh
```

## Using Golang testing

It's also possible to test the developed transactions using Golang testing library. In this type of testing you can simulate a transaction call and compare its results to an expected value.

It's important to note that this type of testing does not work on transactions that reads the chaincode state database (such as using couchdb queries) nor transactions that reads an asset history.

For the cc-tools-demo repository, there are tests for the `createNewLibrary`, `getNumberOfBooksFromLibrary` and `updateBookTenant` transactions.

The tests are defined in the `chaincode/` folder.

```
chaincode/
  main_test.go                                 // setup the test enviroment
  txdefs_createNewLibrary_test.go              // tests the createNewLibrary transaction
  txdefs_getNumberOfBooksFromLibrary_test.go   // tests the getNumberOfBooksFromLibrary transaction
  txdefs_updateBookTenant_test.go              // tests the updateBookTenant transaction

```

### Test examples

The definition of **Create New Library** test is as follows:

```golang
func TestCreateNewLibrary(t *testing.T) {
	stub := mock.NewMockStub("org3MSP", new(cc.CCDemo))

	expectedResponse := map[string]interface{}{
		"@key":         "library:3cab201f-9e2b-579d-b7b2-72297ed17f49",
		"@lastTouchBy": "org3MSP",
		"@lastTx":      "createNewLibrary",
		"@assetType":   "library",
		"name":         "Maria's Library",
	}
	req := map[string]interface{}{
		"name": "Maria's Library",
	}
	reqBytes, err := json.Marshal(req)
	if err != nil {
		t.FailNow()
	}

	res := stub.MockInvoke("createNewLibrary", [][]byte{
		[]byte("createNewLibrary"),
		reqBytes,
	})

	expectedResponse["@lastUpdated"] = stub.TxTimestamp.AsTime().Format(time.RFC3339)

	if res.GetStatus() != 200 {
		log.Println(res)
		t.FailNow()
	}

	var resPayload map[string]interface{}
	err = json.Unmarshal(res.GetPayload(), &resPayload)
	if err != nil {
		log.Println(err)
		t.FailNow()
	}

	if !reflect.DeepEqual(resPayload, expectedResponse) {
		log.Println("these should be equal")
		log.Printf("%#v\n", resPayload)
		log.Printf("%#v\n", expectedResponse)
		t.FailNow()
	}

	var state map[string]interface{}
	stateBytes := stub.State["library:3cab201f-9e2b-579d-b7b2-72297ed17f49"]
	err = json.Unmarshal(stateBytes, &state)
	if err != nil {
		log.Println(err)
		t.FailNow()
	}

	if !reflect.DeepEqual(state, expectedResponse) {
		log.Println("these should be equal")
		log.Printf("%#v\n", state)
		log.Printf("%#v\n", expectedResponse)
		t.FailNow()
	}
}
```

The following steps occur on this test:
- A mock stub for `org3` is created
- The objects with the expected response for the `createNewLibrary` transaction and the transaction request are created
- The `createNewLibrary` transaction is invoked using the request defined on the previous step
- The timestamp from the stub is added to the `expectedResponse` object
- The status of the invoke is checked for the `200` code
- A deep compare is made between the expected response object and the invoke response
- The state for the key of the created object in the stub is also compared to the expected response

The definition of **Get Number Of Books From Library** test is as follows:

```golang
func TestGetNumberOfBooksFromLibrary(t *testing.T) {
	stub := mock.NewMockStub("org2MSP", new(CCDemo))

	// Setup state
	setupBook := map[string]interface{}{
		"@key":         "book:a36a2920-c405-51c3-b584-dcd758338cb5",
		"@lastTouchBy": "org2MSP",
		"@lastTx":      "createAsset",
		"@assetType":   "book",
		"title":        "Meu Nome é Maria",
		"author":       "Maria Viana",
		"genres":       []interface{}{"biography", "non-fiction"},
		"published":    "2019-05-06T22:12:41Z",
	}
	setupLibrary := map[string]interface{}{
		"@key":         "library:3cab201f-9e2b-579d-b7b2-72297ed17f49",
		"@lastTouchBy": "org3MSP",
		"@lastTx":      "createNewLibrary",
		"@assetType":   "library",
		"name":         "Maria's Library",
		"books": []map[string]interface{}{
			{
				"@assetType": "book",
				"@key":       "book:a36a2920-c405-51c3-b584-dcd758338cb5",
			},
		},
	}
	setupBookJSON, _ := json.Marshal(setupBook)
	setupLibraryJSON, _ := json.Marshal(setupLibrary)

	stub.MockTransactionStart("setupGetNumberOfBooksFromLibrary")
	stub.PutState("book:a36a2920-c405-51c3-b584-dcd758338cb5", setupBookJSON)
	stub.PutState("library:3cab201f-9e2b-579d-b7b2-72297ed17f49", setupLibraryJSON)
	refIdx, err := stub.CreateCompositeKey("book:a36a2920-c405-51c3-b584-dcd758338cb5", []string{"library:3cab201f-9e2b-579d-b7b2-72297ed17f49"})
	if err != nil {
		log.Println(err)
		t.FailNow()
	}
	stub.PutState(refIdx, []byte{0x00})
	stub.MockTransactionEnd("setupGetNumberOfBooksFromLibrary")

	expectedResponse := map[string]interface{}{
		"numberOfBooks": 1.0,
	}
	req := map[string]interface{}{
		"library": map[string]interface{}{
			"name": "Maria's Library",
		},
	}
	reqBytes, err := json.Marshal(req)
	if err != nil {
		t.FailNow()
	}

	res := stub.MockInvoke("getNumberOfBooksFromLibrary", [][]byte{
		[]byte("getNumberOfBooksFromLibrary"),
		reqBytes,
	})

	if res.GetStatus() != 200 {
		log.Println(res)
		t.FailNow()
	}

	var resPayload map[string]interface{}
	err = json.Unmarshal(res.GetPayload(), &resPayload)
	if err != nil {
		log.Println(err)
		t.FailNow()
	}

	if !reflect.DeepEqual(resPayload, expectedResponse) {
		log.Println("these should be equal")
		log.Printf("%#v\n", resPayload)
		log.Printf("%#v\n", expectedResponse)
		t.FailNow()
	}
}
```

The following steps occur on this test:
- A mock stub for `org2` is created
- The state objects for the book and the library are defined and added to the mock state on the stub
- The objects with the expected response for the `getNumberOfBooksFromLibrary` transaction and the transaction request are created
- The `getNumberOfBooksFromLibrary` transaction is invoked using the request defined on the previous step
- The status of the invoke is checked for the `200` code
- A deep compare is made between the expected response object and the invoke response

The definition of **Update Book Tenant** test is as follows:

```golang
func TestUpdateBookTenant(t *testing.T) {
	stub := mock.NewMockStub("org1MSP", new(CCDemo))

	// State setup
	setupPerson := map[string]interface{}{
		"@key":         "person:47061146-c642-51a1-844a-bf0b17cb5e19",
		"@lastTouchBy": "org1MSP",
		"@lastTx":      "createAsset",
		"@assetType":   "person",
		"name":         "Maria",
		"id":           "31820792048",
		"height":       0.0,
	}
	setupBook := map[string]interface{}{
		"@key":         "book:a36a2920-c405-51c3-b584-dcd758338cb5",
		"@lastTouchBy": "org2MSP",
		"@lastTx":      "createAsset",
		"@assetType":   "book",
		"title":        "Meu Nome é Maria",
		"author":       "Maria Viana",
		// "currentTenant": map[string]interface{}{
		// 	"@assetType": "person",
		// 	"@key":       "person:47061146-c642-51a1-844a-bf0b17cb5e19",
		// },
		"genres":    []interface{}{"biography", "non-fiction"},
		"published": "2019-05-06T22:12:41Z",
	}
	setupPersonJSON, _ := json.Marshal(setupPerson)
	setupBookJSON, _ := json.Marshal(setupBook)

	stub.MockTransactionStart("setupUpdateBookTenant")
	stub.PutState("person:47061146-c642-51a1-844a-bf0b17cb5e19", setupPersonJSON)
	stub.PutState("book:a36a2920-c405-51c3-b584-dcd758338cb5", setupBookJSON)
	stub.MockTransactionEnd("setupUpdateBookTenant")

	req := map[string]interface{}{
		"book": map[string]interface{}{
			"@key": "book:a36a2920-c405-51c3-b584-dcd758338cb5",
		},
		"tenant": map[string]interface{}{
			"@key": "person:47061146-c642-51a1-844a-bf0b17cb5e19",
		},
	}
	reqBytes, _ := json.Marshal(req)

	res := stub.MockInvoke("updateBookTenant", [][]byte{
		[]byte("updateBookTenant"),
		reqBytes,
	})

	if res.GetStatus() != 200 {
		log.Println(res)
		t.FailNow()
	}

	var resPayload map[string]interface{}
	err := json.Unmarshal(res.GetPayload(), &resPayload)
	if err != nil {
		log.Println(err)
		t.FailNow()
	}

	expectedResponse := map[string]interface{}{
		"@key":         "book:a36a2920-c405-51c3-b584-dcd758338cb5",
		"@lastTouchBy": "org1MSP",
		"@lastTx":      "updateBookTenant",
		"@assetType":   "book",
		"title":        "Meu Nome é Maria",
		"author":       "Maria Viana",
		"currentTenant": map[string]interface{}{
			"@assetType": "person",
			"@key":       "person:47061146-c642-51a1-844a-bf0b17cb5e19",
		},
		"genres":    []interface{}{"biography", "non-fiction"},
		"published": "2019-05-06T22:12:41Z",
	}

	expectedResponse["@lastUpdated"] = stub.TxTimestamp.AsTime().Format(time.RFC3339)

	if !reflect.DeepEqual(resPayload, expectedResponse) {
		log.Println("these should be equal")
		log.Printf("%#v\n", resPayload)
		log.Printf("%#v\n", expectedResponse)
		t.FailNow()
	}

	var state map[string]interface{}
	stateBytes := stub.State["book:a36a2920-c405-51c3-b584-dcd758338cb5"]
	err = json.Unmarshal(stateBytes, &state)
	if err != nil {
		log.Println(err)
		t.FailNow()
	}

	if !reflect.DeepEqual(state, expectedResponse) {
		log.Println("these should be equal")
		log.Printf("%#v\n", state)
		log.Printf("%#v\n", expectedResponse)
		t.FailNow()
	}
}
```

The following steps occur on this test:
- A mock stub for `org1` is created
- The state objects for the book and the person are defined and added to the mock state on the stub
- The object with the request for the `updateBookTenant` transaction is created
- The status of the invoke is checked for the `200` code
- The expected response object is created, with the stub timestamp added to the `@lastUpdated` value
- A deep compare is made between the expected response object and the invoke response
- The state for the key of the book in the stub is also compared to the expected response

### Running the tests

To run the tests, execute the following command:

```sh
cd chaincode && \
go test && \
cd ..
```

If the tests were succeseful, the following messages should appear on the terminal:

```sh
main.go:131: 200 init 40.796µs 
main.go:131: 200 getNumberOfBooksFromLibrary 31.298µs 
main.go:131: 200 updateBookTenant 136.09µs 
main.go:131: 200 createNewLibrary 60.794µs 
PASS
ok      github.com/goledgerdev/cc-tools-demo/chaincode  0.006s
```
