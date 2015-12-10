## The Message object

HTTP is a stream protocol. Web sockets are frame-based. You prepare a block of data (of any size) and send it as a set of frames. Frames can contain either strings in UTF-8 encoding or a sequence of bytes.

The simplest way of using web sockets is just to prepare a block of data and ask the Go websocket library to package it as a set of frame data, send them across the wire and receive it as the same block. The `websocket` package contains a convenience object `Message` to do just that. The `Message` object has two methods, `Send` and `Receive` which take a websocket as first parameter. The second parameter is either the address of a variable to store data in, or the data to be sent. Code to send string data would look like

```go
msgToSend := "Hello"
err := websocket.Message.Send(ws, msgToSend)

var msgToReceive string
err := websocket.Message.Receive(conn, &msgToReceive)
```

Code to send byte data would look like

```go
dataToSend := []byte{0, 1, 2}
err := websocket.Message.Send(ws, dataToSend)

var dataToReceive []byte
err := websocket.Message.Receive(conn, &dataToReceive)
```

An echo server to send and receive string data is given below. Note that in web sockets either side can initiate sending of messages, and in this server we send messages from the server to a client when it connects (send/receive) instead of the more normal receive/send server. The server is

```go
/* EchoServer
 */
package main

import (
	"fmt"
	"net/http"
	"os"
	// "io"
	"golang.org/x/net/websocket"
)

func Echo(ws *websocket.Conn) {
	fmt.Println("Echoing")

	for n := 0; n < 10; n++ {
		msg := "Hello  " + string(n+48)
		fmt.Println("Sending to client: " + msg)
		err := websocket.Message.Send(ws, msg)
		if err != nil {
			fmt.Println("Can't send")
			break
		}

		var reply string
		err = websocket.Message.Receive(ws, &reply)
		if err != nil {
			fmt.Println("Can't receive")
			break
		}
		fmt.Println("Received back from client: " + reply)
	}
}

func main() {

	http.Handle("/", websocket.Handler(Echo))
	err := http.ListenAndServe(":12345", nil)
	checkError(err)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

A client that talks to this server is

```go
/* EchoClient
 */
package main

import (
	"golang.org/x/net/websocket"
	"fmt"
	"io"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "ws://host:port")
		os.Exit(1)
	}
	service := os.Args[1]

	conn, err := websocket.Dial(service, "", "http://localhost")
	checkError(err)
	var msg string
	for {
		err := websocket.Message.Receive(conn, &msg)
		if err != nil {
			if err == io.EOF {
				// graceful shutdown by server
				break
			}
			fmt.Println("Couldn't receive msg " + err.Error())
			break
		}
		fmt.Println("Received from server: " + msg)
		// return the msg
		err = websocket.Message.Send(conn, msg)
		if err != nil {
			fmt.Println("Coduln't return msg")
			break
		}
	}
	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

The url for the client running on the same machine as the server should be `ws://localhost:12345/`

