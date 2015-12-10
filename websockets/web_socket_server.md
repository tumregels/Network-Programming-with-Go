## Web socket server

 A web socket server starts off by being an HTTP server, accepting TCP conections and handling the HTTP requests on the TCP connection. When a request comes in that switches that connection to a being a web socket connection, the protocol handler is changed from an HTTP handler to a WebSocket handler. So it is only that TCP connection that gets its role changed: the server continues to be an HTTP server for other requests, while the TCP socket underlying that one connection is used as a web socket.

One of the simple servers HHTP we discussed in Chapter 8: HTTP registered varous handlers such as a file handler or a function handler. To handle web socket requests we simply register a different type of handler - a web socket handler. Which handler the server uses is based on the URL pattern. For example, a file handler might be registered for "/", a function handler for "/cgi-bin/..." and a web sockets handler for "/ws".

An HTTP server that is only expecting to be used for web sockets might run by

```go
func main() {
    http.Handle("/", websocket.Handler(WSHandler))
    err := http.ListenAndServe(":12345", nil)
    checkError(err)
}
```

A more complex server might handle both HTTP and web socket requests simply by adding in more handlers. 


