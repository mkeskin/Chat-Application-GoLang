# Go Chat Application with Angular
Chat Application written by [root](http://root.ist) and used Golang and AngularJS. Thanks to @vinceprignano.

### Installation of socket.io
Select the Gopath:
```
export GOPATH=YOUR_PROJECT_DIR
```

Get the socket.io package:
```
go get github.com/googollee/go-socket.io
```

### Usage of application
Use `go run` to start:
```
go run chat.go
```

### Details of Go codes

###### Starting of socket.io
```
sio.On("connection", func(so socketio.Socket) {
  ...
}
```

###### Listening of new message on socket
```
so.On("send_message", func(message string) {
  ...
}
```

###### Sending a copy of new message to Netdata.com
Don't forget to change YOUR_API_KEY. You can have a API Key on Netdata.com for project or can using default key. If is set new message, sending to socket broatcast and Netdata.com for saving message.
```
log.Printf("Sent message from %s: %s", user, message)

res := map[string]interface{} {
	"username": user,
	"message":  message,
	"dateTime": time.Now().UnixNano() / int64(time.Millisecond),
	"type":     "message",
}

jsonRes, _ := json.Marshal(res)

lmMutex.Lock()
if len(lastMessages) == 100 {
	lastMessages = lastMessages[1:100]
}
lastMessages = append(lastMessages, string(jsonRes))
lmMutex.Unlock()

so.Emit("message", string(jsonRes))
so.BroadcastTo(websocketRoom, "message", string(jsonRes))

soapRequestContent := "<?xml version=\"1.0\" encoding=\"utf-8\"?><soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\"><soap:Body><InsertRecord xmlns=\"http://tempuri.org/\"><APIKey>YOUR_API_KEY</APIKey><InsertList><AccPoKeyValuePair><Key>dc_UserName</Key><Value>"+user+"</Value></AccPoKeyValuePair><AccPoKeyValuePair><Key>dc_Message</Key><Value>"+message+"</Value></AccPoKeyValuePair><AccPoKeyValuePair><Key>dc_MessageTo</Key><Value>all</Value></AccPoKeyValuePair></InsertList></InsertRecord></soap:Body></soap:Envelope>"

httpClient := new(http.Client)
_, err := httpClient.Post("http://www.netdata.com/AccPo.asmx", "text/xml; charset=utf-8", bytes.NewBufferString(soapRequestContent))

if err != nil {
  log.Fatal(err)
}
```

###### Changing of application port
Can change default port quikly.
```
listen := os.Getenv("LISTEN")

if listen == "" {
  listen = ":3001"
}
```
