# Events

Hyperledger Fabric allows client applications (such as the rest-server API) to receive block events while block are commited to the peer's ledger.

CC-Tools have a built-in event funcionality that allows for a quick register of the event, which will automatically be listened and handled by the standard CCAPI.

The events in CC-Tools can be of three types: `log`, registering the information sent in the event payload on the CCAPI logs; `transaction`, which invokes another chaincode transactions when the event is received; and `custom`, which executes a custom function previously defined in the event.

For the cc-tools-demo repository, there are 1 pre-defined event registered:

- Create Library Log

The events definition is done in the **chaincode/eventtypes** folder

Here follow the list of files:

    chaincode/
        eventtypes/     		  # event folder
            createLibraryLog.go   # definition of the library log event
    eventTypeList.go   			  # list of events instantiated

## Event definition

The construction and definition of an event is done by creating a file at **chaincode/eventtypes** folder

An event has the following fields

- `Tag`: string field used to define the name of the event referenced internally by the code and by the rest-server clients.
- `Label`: string field to define the label to be used by external applications and the rest-server clients. Free text.
- `Description`: asset description string field to be used by external applications. Free text.
- `BaseLog`: string field with a message to be logged on the CCAPI at every event calling. Free text. 
- `EventType`: defines the type of the event. Possible options being `events.EventLog`, `events.EventTransaction` and `events.EventCustom`.
- `Transaction`: string field with the tag of the transaction to be called by events of the `transaction` type.
- `Channel`: string field with the channel name of where the transaction to be called by an event of the `transaction` type is located. If empty, defaults to the same channel of the caller.
- `Chaincode`: string field with the chaincode name of where the transaction to be called by an event of the `transaction` type is located. If empty, defaults to the same chaincode of the caller.
- `CustomFunction`: a custom function to be executed by events of the `custom` type. It receives a stub of the type `*sw.StubWrapper` and a payload of type `[]byte`, sent alongside the event, and returns an error.
- `ReadOnly`: a boolean which defines if the custom functions for events of type `custom` can alter the world state or not. If `true`, no assets will be able to be commited to the ledger by the custom function.


## Event example

The `cc-tools-demo` repository has the following example:

    chaincode/
        eventtypes/     		  # event folder
            createLibraryLog.go   # definition of the library log event
    eventTypeList.go    		  # list of events instantiated

Besides the files of each event, you must register the events that can be used by the **Goledger CC-Tools** library in the **eventTypeList.go** file

The definition of the **CreateLibraryLog** asset is as follows:

```golang
var CreateLibraryLog = events.Event{
	Tag:         "createLibraryLog",
	Label:       "Create Library Log",
	Description: "Log of a library creation",
	Type:        events.EventLog,                 // Event funciton is to log on the CCAPI
	BaseLog:     "New library created",           // BaseLog is a base message to be logged
	Receivers:   []string{"$org2MSP", "$orgMSP"}, // 	Receivers are the MSPs that will receive the event
}
```

According to the description above, the event **CreateLibraryLog** has the following characteristics:

- Is of `log` type, registering its messages on the CCAPI logs
- Every event of this type received will have the base message `New library created` logged on the CCAPI 
- Only **org2** and **org** will be registered to wait for events of this type

## Event list definition

**GoLedger CC-Tools** events registration must be defined in the **chaincode/eventTypeList.go** file

```golang
var eventTypeList = []events.Event{
	eventtypes.CreateLibraryLog,
}
```

## Calling the events

CC-Tools event can be called from the transactions using the stub object provided and a `[]byte` can be sent alongside it.

For `log` events, the payload is a marshalled string:

```golang
payload, ok := json.Marshal("the event payload")
```

For `transaction` events, the payload is a marshalled object containing the parameters for the called transaction:

```golang
payload, ok := json.Marshal(map[string]interface{}{
	"library": libraryAsset,
})
```

For `custom` events the payload can be anything on the `[]byte` format, to be handled accordingly by the custom function.

It's possible to call events using the event object:

```golang
eventObj.CallEvent(stub, payload)
```

or using the event tag to trigger it:

```golang
events.CallEvent(stub, "createLibraryLog", payload)
```
