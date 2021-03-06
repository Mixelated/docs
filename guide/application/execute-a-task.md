# Execute a task

## Why execute a Service's task?

Applications can execute a Service's task, allowing you to reuse the maximum number of already-built logic and enjoy the MESG ecosystem.

## Execute a Service's task

To execute a task, Applications need to connect to Core through [gRPC](https://grpc.io/) and use the [Protobuffer definition](https://github.com/mesg-foundation/core/blob/master/protobuf/coreapi/api.proto). Core will reply with an `executionID`that identifies the task's execution. To get the output of the task's execution, the Application has to listen for an [execution output.](./listen-for-events.md#listen-for-task-execution-outputs)

<tabs>
<tab title="Request" vp-markdown>

### `Core.ExecuteTask`

| **Name** | **Type** | **Required** | **Description** |
| --- | --- | --- | --- |
| **serviceId** | `String` | Required | ID of the service. |
| **taskKey** | `String` | Required | The task's key defined in the [service file](../service/service-file.md). |
| **inputData** | `String` | Required | The task's inputs in JSON format. |
| **executionTags** | `Array<String>` | Optional | The list of tags to associate with the execution. |

```javascript
{
  "serviceID": "f4923d9de32f211a1e3fbd54399752c305e2db72",
  "taskKey": "taskX",
  "inputData": "{\"inputX\":\"input value\"}",
  "executionTags": ["tagX=1", "tagY"]
}
```

</tab>

<tab title="Reply" vp-markdown>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| **executionID** | `String` | The ID of the execution. |

```javascript
{
  "executionID": "xxxxx"
}
```

</tab>
</tabs>

### Examples

<tabs>
<tab title="Node" vp-markdown>

```javascript
const mesg = require('mesg-js').application()

mesg.executeTask({
  serviceID: SERVICE_TASK_ID,
  taskKey: "taskX",
  inputData: JSON.stringify({
    inputX: "input value"
  })
}).then((reply) => {
  console.log(reply.executionID)
}).catch((err) => {
  console.error('an error occurred while executing task:', err.message)
})
```

[See the MESG.js library for additional documentation](https://github.com/mesg-foundation/mesg-js/tree/master#execute-task)

</tab>

<tab title="Go" vp-markdown>

```go
package main

import (
	"context"
	"encoding/json"
	"log"

	"github.com/mesg-foundation/core/protobuf/coreapi"
	"google.golang.org/grpc"
)

func main() {
	connection, err := grpc.Dial(":50052", grpc.WithInsecure())
	if err != nil {
		log.Panic(err)
	}
	client := coreapi.NewCoreClient(connection)

  inputData, err := json.Marshal(map[string]string{"foo": "bar"})
  if err != nil {
    log.Panic(err)
  }
  reply, err := client.ExecuteTask(context.Background(), &coreapi.ExecuteTaskRequest{
    ServiceID: "SERVICE_TASK_ID",
    TaskKey:   "taskX",
    InputData: string(inputData),
  })
  if err != nil {
    log.Panic(err)
  }
  log.Println("execution reply", reply)
}

```

[See the Core API for additional documentation](https://docs.mesg.com/api/core.html#core-api)

</tab>
</tabs>

::: tip Get Help
You need help ? Check out the <a href="https://forum.mesg.com" target="_blank">MESG Forum</a>.
