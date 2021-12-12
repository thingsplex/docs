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
Currenlty the runtime supports interpreted version of `golang` . Descpite having the same syntax,it has limited access to external libraries and is less performant compare to normal compiled golang compiler. Golang is very simple yet powerfull language - <https://golang.org/>

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
    Timeseries  *timeseries.Connector
    Settings    map[string]model.Setting
    Log         *log.Entry
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
github.com/thingsplex/tpflow/connector/plugins/timeseries
github.com/futurehomeno/fimpgo
```

### Time series API

Functions :

```golang

func (conn *Connector) QueryDataPoints(query string) (*Result,error)

func (conn *Connector) GetDataPoints(request *GetDataPointsRequest) (*Result,error) 

func (conn *Connector) GetEnergyDataPoints(request *GetDataPointsRequest) (*Result,error)

func (conn *Connector) WriteDataPoints(request *WriteDataPointsRequest) error

```

Types :

```golang

type DataPointsFilter struct {
 Tags      map[string]string `json:"tags"`
 Devices   []string          `json:"devices"`
 Locations []string          `json:"locations"`
 DevTypes  []string          `json:"dev_types"`
}

type GetDataPointsRequest struct {
 ProcID            int              `json:"proc_id"`
 FieldName         string           `json:"field_name"`
 DataFunction      string           `json:"data_function"`
 TransformFunction string           `json:"transform_function"`
 MeasurementName   string           `json:"measurement_name"`
 RelativeTime      string           `json:"relative_time"`
 FromTime          string           `json:"from_time"` // 2021-10-24T00:00:00Z
 ToTime            string           `json:"to_time"`   // 2021-10-27T00:00:00Z
 GroupByTime       string           `json:"group_by_time"`
 GroupByTag        string           `json:"group_by_tag"`
 FillType          string           `json:"fill_type"`
 Filters           DataPointsFilter `json:"filters"`
}

type WriteDataPointsRequest struct {
 ProcID     int          `json:"proc_id"`
 Bucket     string       `json:"bucket"` // data is stored with retention policy , if not set system will try to auto calculate based on measurement name
 DataPoints []MDataPoint `json:"dp"`
}

type MDataPoint struct {
 Name      string                 `json:"name"` // name of the measurement
 Tags      map[string]string      `json:"tags"`
 Fields    map[string]interface{} `json:"fields"`
 TimeStamp int64                  `json:"ts"` // if 0 , ecollector will set local time
}

type Result struct {
 Series   []Row
 Err      string `json:"error,omitempty"`
}

// Row represents a single row returned from the execution of a statement.
type Row struct {
 Name    string            `json:"name,omitempty"`
 Tags    map[string]string `json:"tags,omitempty"`
 Columns []string          `json:"columns,omitempty"`
 Values  [][]interface{}   `json:"values,omitempty"`
 Partial bool              `json:"partial,omitempty"`
}


```

Example JSON response :

```json

[
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "10"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                null
            ],
            [
                1635242400,
                21.5
            ],
            [
                1635246000,
                22.5
            ]
        ]
    },
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "11"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                22
            ],
            [
                1635242400,
                22.600000381469727
            ],
            [
                1635246000,
                22.5
            ]
        ]
    },
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "12"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                23.100000381469727
            ],
            [
                1635242400,
                23.68
            ],
            [
                1635246000,
                23.6
            ]
        ]
    },
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "13"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                20.17
            ],
            [
                1635242400,
                20.17
            ],
            [
                1635246000,
                20.17
            ]
        ]
    },
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "16"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                8.199999809265137
            ],
            [
                1635242400,
                8.899999618530273
            ],
            [
                1635246000,
                9.800000190734863
            ]
        ]
    },
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "7"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                3
            ],
            [
                1635242400,
                22.3
            ],
            [
                1635246000,
                22.3
            ]
        ]
    },
    {
        "name": "sensor_temp.evt.sensor.report",
        "tags": {
            "location_id": "8"
        },
        "columns": [
            "time",
            "value"
        ],
        "values": [
            [
                1635238800,
                22.58
            ],
            [
                1635242400,
                21.1200008392334
            ],
            [
                1635246000,
                21.0699996948242
            ]
        ]
    }
]

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

 params.Log.Info("New request , counter = ",counter)

 ctx.SetVariable("test_var","string",r,"",params.FlowId,true)
 ctx.SetVariable("request_var","object",msg.ValPayload,"",params.FlowId,true)
 return "ok"
}

```

```golang

package ext

import "fmt"
import "github.com/thingsplex/tpflow/model"
import "github.com/thingsplex/tpflow/node/action/exec"
import "github.com/futurehomeno/fimpgo"
import "github.com/thingsplex/tpflow/connector/plugins/timeseries"

var counter int = 1

type AppLogic struct {
    mqtt *fimpgo.MqttTransport
    ctx *model.Context
}

func sendDataPoint(params exec.ScriptParams) {
	dp := timeseries.MDataPoint{
		Name    : "test_data_point",
		Tags    : map[string]string {},
		Fields  :  map[string]interface{} {"val":10},
	}

	dpReq := timeseries.WriteDataPointsRequest{
		ProcID     :1,
		Bucket     :"gen_default",
		DataPoints :[]timeseries.MDataPoint{dp},
	   }   
	   
	params.Timeseries.WriteDataPoints(&dpReq)
}

func Run(msg *model.Message,ctx *model.Context,params exec.ScriptParams) string {
//  appL := AppLogic{mqtt:params.Mqtt,ctx:ctx}
 
 tsReq := timeseries.GetDataPointsRequest{
    ProcID            :1,
	FieldName         :"value",
	DataFunction      :"last",
	MeasurementName   :"sensor_temp.evt.sensor.report",
	RelativeTime      :"12h",
	GroupByTime       :"1h",
	GroupByTag        :"location_id",
	FillType          :"previous"}
 tsResult,err := params.Timeseries.GetDataPoints(&tsReq)     
 if err != nil {
     params.Log.Error("Error :",err)
 } else {
     msg.ValPayload = tsResult.Series
 }

 sendDataPoint(params)

 params.Log.Info("New request , counter = ",counter)
 
 counter++
//  r := fmt.Sprintf("Hello %d",counter)
//  ctx.SetVariable("hello_var","string",r,"",params.FlowId,true)
 return "ok"
}

```

Parsing HTTP request , requesting time series data and sending response : 

```golang

package ext

import "fmt"
import "encoding/json"
import "github.com/thingsplex/tpflow/model"
import "github.com/thingsplex/tpflow/node/action/exec"
import "github.com/futurehomeno/fimpgo"
import "github.com/thingsplex/tpflow/connector/plugins/timeseries"

var counter int = 1

func Run(msg *model.Message,ctx *model.Context,params exec.ScriptParams) string {

 tsReq := timeseries.GetDataPointsRequest{
    ProcID            :1,
	FieldName         :"value",
	DataFunction      :"last",
	MeasurementName   :"sensor_temp.evt.sensor.report",
	RelativeTime      :"3d",
	GroupByTime       :"1h",
	GroupByTag        :"location_id",
	FillType          :"previous"}
 
 params.Log.Info("Input payload size = ",len(msg.RawPayload))
 
 // Unmarshaling Flowframe variable int tsReq variable 
 json.Unmarshal(msg.RawPayload,&tsReq) 	

 tsResult,err := params.Timeseries.GetDataPoints(&tsReq)     
 if err != nil {
     params.Log.Error("Error :",err)
 } else {
     msg.ValPayload = tsResult.Series
 }
 params.Log.Info("New request , counter = ",counter)
 
 counter++
//  ctx.SetVariable("hello_var","string",r,"",params.FlowId,true)
 return "ok"
}


```

