# satellite  <img src="https://image.flaticon.com/icons/svg/917/917274.svg" width="48">

# Fast and simple JSON processing TCP server written in Go

Designed to act as a template for server side JSON processing.
Simply modify the global MyObject struct with the fields you need, run the server, and import the the API into any package.

# usage
`go get github.com/trashbo4t/satellite`

`import "github.com/trashbo4t/satellite"`

```
Usage of satellite:
  -d    set log level to debug
  -e    set log level to error
  -i    set log level to info
  -s    output to stderr
  -t    listen on TCP (default true)
  -u    listen on UDP
  -w    set log level to warning
```

The API will send the serialized struct to the server, which will accept the connection, handle it on a seperate thread, inject a response, and send data back all from one call:

` conn, err := satellite.SendJson(obj) `

The TCP server works as follows:

    - satellite.go
	- A socket is opened and listened on the main thread of execution
	- 5 TcpHandlers run as go routines (concurrently) and listen on a channel for incoming connections.
	- Main thread receives connections and sends them down the channel.
	- Any TcpHandler picks up the connection request and Marshals the json message.
	- If an error occurs the the socket is closed, and a RST packet is sent to the user.

Only a couple of changes are required to customize the library for your needs

* object.go
```
type MyObject struct {
	Magic uint8
	Msg   string
	Response string
}
```

* common.go 
```
// HandleJSON
// Modify this function to do whatever you need to do
// with your struct.
func HandleJSON(b []byte) ([]byte, error) {
	obj, err := AsJson(b)
	if err != nil {
		return nil, err
	}
	// Do what you will with the JSON object here
	// Inject a response into the struct and send it back
	obj.Response = "Handled"
	// ...
	// ...
	// ...
	return AsByte(obj)
}
```


# Example

Start the server with multiple listeners.
```
$ go run satellite.go -d -t -s
2018-05-27T15:07:21-04:00 |INFO| Started Satellite "Beep Boop Bop Beep" 
2018-05-27T15:07:21-04:00 |INFO| Listening on localhost:9876
2018-05-27T15:07:21-04:00 |DEBU| Launching TCP Handlers
2018-05-27T15:07:21-04:00 |DEBU| TCP handler is up
2018-05-27T15:07:21-04:00 |DEBU| TCP handler is up
2018-05-27T15:07:21-04:00 |DEBU| TCP handler is up
2018-05-27T15:07:21-04:00 |DEBU| TCP handler is up
2018-05-27T15:07:21-04:00 |DEBU| TCP handler is up
```

Run the example client 
```
$ go run exampleclient.go
```

```
2018-05-27T15:07:26-04:00 |DEBU| Accepted incoming connection
2018-05-27T15:07:26-04:00 |DEBU| Received Message: {"Magic":1,"Msg":"This is a test message","Response":""}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
2018-05-27T15:07:26-04:00 |DEBU| Total bytes: 56
2018-05-27T15:07:26-04:00 |DEBU| Sending reply: {"Magic":1,"Msg":"This is a test message","Response":"Handled"}
```

```
{"level":"info","time":1527448045,"message":"This is an example test of the satellite Json server"}
{"level":"info","time":1527448046,"message":"Object sent to the server without error"}
{"level":"info","time":1527448046,"message":"Received 63 bytes from satellite"}
{"level":"info","time":1527448046,"message":"Json response from satellite: Handled"}
```
