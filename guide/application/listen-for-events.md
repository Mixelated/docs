# Listen for events

## Why listen for events?

Applications listen for events in real-time to execute actions or tasks when something relevant happens on a technology.

## Listening for events from Services

To listen for events, the Application needs to open a stream with Core with [gRPC](https://grpc.io/) using the [Protobuffer definition](https://github.com/mesg-foundation/core/blob/master/protobuf/coreapi/api.proto). When opening the stream, the Application listens to the Service. It can listen to many Services at the same time.

<tabs>
<tab title="Request" vp-markdown>

### `Core.ListenEvent`

| **Name** | **Type** | **Required** | **Description** |
| --- | --- | --- | --- |
| **serviceID** | `String` | Required | ID of the Service that you want to listen to. |
| **eventFilter** | `String` | Optional | Only listens for this given event ID. |

```javascript
{
  "serviceID": "f4923d9de32f211a1e3fbd54399752c305e2db72",
  "eventFilter": "eventIDToOnlyListenTo"
}
```

</tab>

<tab title="Stream Reply" vp-markdown>

| **Name** | **Type** | **Description** |
| --- | --- | --- | --- |
| **eventKey** | `String` | The event's key defined in the [Service file](../service/service-file.md). |
| **eventData** | `String` | The event's data in JSON format. |

```javascript
{
  "eventKey": "eventX",
  "eventData": "{\"dataX\": \"event data\"}"
}
```

</tab>
</tabs>

### Example

<tabs>
<tab title="Node" vp-markdown>

```javascript
const mesg = require('mesg-js').application()

mesg.listenEvent({
  serviceID: SERVICE_EVENT_ID
})
  .on('data', (event) => {
    console.log('event received', event)
  })
  .on('error', (err) => {
    console.error('an error occurred while listening for events:', err.message)
  })

console.log('application is running and listening for events')
```

[See the MESG.js library for additional documentation](https://github.com/mesg-foundation/mesg-js/tree/master#listen-events)

</tab>

<tab title="Go" vp-markdown>

```go
package main

import (
	"context"
	"encoding/json"
	"log"

	"github.com/mesg-foundation/core/protobuf/coreapi"
	"github.com/mesg-foundation/core/x/xsignal"
	"google.golang.org/grpc"
)

func main() {
	connection, err := grpc.Dial(":50052", grpc.WithInsecure())
	if err != nil {
		log.Panic(err)
	}
	defer func() {
		log.Println("closing connection...")
		connection.Close()
	}()
	client := coreapi.NewCoreClient(connection)
	log.Println("connected to core")

	go func() {
		stream, err := client.ListenEvent(context.Background(), &coreapi.ListenEventRequest{
			ServiceID:   "SERVICE_EVENT_ID",
			EventFilter: "eventX", // optional
		})
		if err != nil {
			log.Panic(err)
		}

		for {
			event, err := stream.Recv()
			if err != nil {
				log.Panic(err)
			}
			log.Println("event received", event)
		}
	}()

	<-xsignal.WaitForInterrupt()
}

```

[See the Core API for additional documentation](https://docs.mesg.com/api/core.html#core-api)

</tab>
</tabs>

## Listen for task execution outputs

The execution of tasks can take a long time to finish depending on the action they are completing, so outputs are sent back asynchronously. To listen for task execution's outputs, applications need to open a stream with Core through [gRPC](https://grpc.io/) and use the [Protobuffer definition](https://github.com/mesg-foundation/core/blob/master/protobuf/coreapi/api.proto).

::: warning
Outputs are sent asynchronously. Make sure that the Application listens for outputs before it executes a task, otherwise it will miss the outputs.
:::

<tabs>
<tab title="Request" vp-markdown>

### `Core.ListenResult`

| **Name** | **Type** | **Required** | **Description** |
| --- | --- | --- | --- |
| **serviceID** | `String` | Required | ID of the Service. |
| **taskFilter** | `String` | Optional | Only listens for this given task ID. |
| **outputFilter** | `String` | Optional | Only listens for this given output ID. If set, the attribute `taskFilter` should also be provided. |
| **tagFilters** | `Array<String>` | Optional | The list of tags to filter. This is a "match all" list. All tags in parameters should be included in the execution to match. |

```javascript
{
  "serviceID": "f4923d9de32f211a1e3fbd54399752c305e2db72",
  "taskFilter": "taskIDToOnlyListenTo",
  "outputFilter": "outputIDToOnlyListenTo",
  "tagFilters": ["tagX=1"]
}
```

</tab>

<tab title="Stream Reply" vp-markdown>

| **Name** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- |
| **executionID** | `String` | The execution ID of this output. |
| **taskKey** | `String` | The key of the task as defined in the [service file](../service/service-file.md). |
| **outputKey** | `String` | The key of the output of the task as defined in the [service file](../service/service-file.md). |
| **outputData** | `String` | The data returned by the task serialized in JSON. |

```javascript
{
  "executionID": "xxxxx",
  "taskKey": "taskX",
  "outputKey": "outputX",
  "outputData": "{\"outputValX\": \"result of execution\"}",
}
```

</tab>
</tabs>

### Examples

<tabs>
<tab title="Node" vp-markdown>

```javascript
const mesg = require('mesg-js').application()

mesg.listenResult({
  serviceID: SERVICE_RESULT_ID
})
  .on('data', (result) => {
    console.log('result received', result)
  })
  .on('error', (err) => {
    console.error('an error occurred while listening for results:', err.message)
  })

console.log('application is running and listening for results')
```

[See the MESG.js library for additional documentation](https://github.com/mesg-foundation/mesg-js/tree/master#listen-results)

</tab>

<tab title="Go" vp-markdown>

```go
package main

import (
	"context"
	"encoding/json"
	"log"

	"github.com/mesg-foundation/core/protobuf/coreapi"
	"github.com/mesg-foundation/core/x/xsignal"
	"google.golang.org/grpc"
)

func main() {
	connection, err := grpc.Dial(":50052", grpc.WithInsecure())
	if err != nil {
		log.Panic(err)
	}
	defer func() {
		log.Println("closing connection...")
		connection.Close()
	}()
	client := coreapi.NewCoreClient(connection)
	log.Println("connected to core")

	go func() {
		stream, err := client.ListenResult(context.Background(), &coreapi.ListenResultRequest{
			ServiceID:  "SERVICE_TASK_ID",
			TaskFilter: "taskX", // optional
		})
		if err != nil {
			log.Panic(err)
		}

		for {
			result, err := stream.Recv()
			if err != nil {
				log.Panic(err)
			}
			log.Println("result received", result)
		}
	}()

	<-xsignal.WaitForInterrupt()
}

```

[See the Core API for additional documentation](https://docs.mesg.com/api/core.html#core-api)

</tab>
</tabs>

::: tip Get Help
You need help ? Check out the <a href="https://forum.mesg.com" target="_blank">MESG Forum</a>.
