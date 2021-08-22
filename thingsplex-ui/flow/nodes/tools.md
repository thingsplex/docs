---
layout: default
title: Tools
parent: Flows
grand_parent : Thingsplex UI
nav_order: 2
---

# Run script action 



## Custom script 

Scripting enables a user to implement complex logic or complex transformations. 
Currenlty the runtime supports interpreted version of `golang` . Descpite having the same syntax,it has limited access to external libraries and is less performant compare to normal compiled golang compiler. Golang is very simple yet powerfull language - https://golang.org/

User scrip is executed in `Run script` node. Each scripting node keeps states of all internal variables untill flow is reloaded or flow engine is reastarted.

### Main handler 

Main message handler is invoked by runtime once the node receives message from previous node.
Handler function implemtn the signature : 

```golang
func Run(msg *model.Message,ctx *model.Context,params exec.ScriptParams) string {
    return "ok"
}
```

Parameters :

`msg *model.Message` - is a reference to trigger message object. 

```golang
type Message struct {
    AddressStr string
    Address    fimpgo.Address
    Payload    fimpgo.FimpMessage
    ValPayload interface{} // the field is optional. Must be used to generate response if is set , if not set , Payload must be used instead
    RawPayload []byte
    Header     map[string]string
    CancelOp   bool // if true , listening node should close all operations
    RequestId  int64
    Origin     string
}
```

```golang
type ScriptParams struct {
    FlowId      string
    Mqtt        *fimpgo.MqttTransport
    Registry    storage.RegistryStorage
    Settings    map[string]model.Setting
}
```

Return :

If function returns `ok` the node follows successfull path in transitions to next node otherwise it follows error transition path. 


### List of supported libraries 

Golang stdlib :
```
fmt
time
strconv
strings
math
sort
encoding/json
net
net/http

```

Extension libraries : 

```
github.com/thingsplex/tpflow/model
github.com/thingsplex/tpflow/node/action/exec
github.com/thingsplex/tpflow/registry/storage
github.com/futurehomeno/fimpgo
```


### Examples

```golang

package ext

import "fmt"
import "github.com/thingsplex/tpflow/model"
import "github.com/thingsplex/tpflow/node/action/exec"
import "github.com/futurehomeno/fimpgo"

var counter int = 1

type AppLogic struct {
    mqtt *fimpgo.MqttTransport
    ctx *model.Context
}

func(al *AppLogic) sendMessage(msg string) {
    fimpMsg := fimpgo.NewMessage("evt.script.test","test","string",msg,nil,nil,nil)
    al.mqtt.PublishToTopic("pt:j1/mt:evt/rt:app/rn:testapp/ad:1",fimpMsg)
    
}

func(al *AppLogic) CheckHomeMode() {
    hModeVal,err := al.ctx.GetVariable("fh.home.mode","global")
    if err != nil {
        return 
    }
    hMode, ok := hModeVal.Value.(string)
    if !ok {
        return
    }
    if hMode == "home" {
        al.sendMessage("home is set to home mode")
    }
}

func Run(msg *model.Message,ctx *model.Context,params exec.ScriptParams) string {
 appL := AppLogic{mqtt:params.Mqtt,ctx:ctx}
 appL.CheckHomeMode()
 
 counter++
 r := fmt.Sprintf("Hello %d",counter)
 ctx.SetVariable("hello_var","string",r,"",params.FlowId,true)
 return "ok"
}

```

```golang
package ext

import "fmt"

import "github.com/thingsplex/tpflow/model"
import "github.com/thingsplex/tpflow/node/action/exec"

var counter int = 1

type TestMsg struct {
    Name string 
    Counter int
}

type Response struct {
    Result string 
    ErrorCode string
}

func Run(msg *model.Message,ctx *model.Context,params exec.ScriptParams) string {
 var request TestMsg
 msg.PayloadMsgToStruct(&request) // cast request object to variable
 counter = request.Counter
 
 // mutating trigger request variable  
 msg.ValPayload = Response {
     Result:"ok",
 }
 r := fmt.Sprintf("Hello %d",counter)
 ctx.SetVariable("test_var","string",r,"",params.FlowId,true)
 ctx.SetVariable("request_var","object",msg.ValPayload,"",params.FlowId,true)
 return "ok"
}

```
