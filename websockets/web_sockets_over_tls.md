## Web sockets over TLS

A web socket can be built above a secure TLS socket. We discussed in [Chapter 8: HTTP](../http/README.md) how to use a TLS socket using the certificates from [Chapter 7: Security](../security/README.md). That is used unchanged for web sockets. that is, we use `http.ListenAndServeTLS` instead of `http.ListenAndServe`.

Here is the echo server using TLS

```go
/* EchoServer
 */
package main

import (
	"golang.org/x/net/websocket"
	"fmt"
	"net/http"
	"os"
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
	err := http.ListenAndServeTLS(":12345", "jan.newmarch.name.pem",
		"private.pem", nil)
	checkError(err)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

The client is the same echo client as before. All that changes is the url, which uses the `"wss"` scheme instead of the `"ws"` scheme:

```
EchoClient wss://localhost:12345/
```

## Conclusion

The web sockets standard is nearing completion and no major changes are anticipated. This will allow HTTP user agents and servers to set up bi-directional socket connections and should make certain interaction styles much easier. Go has nearly complete support for web sockets. 



