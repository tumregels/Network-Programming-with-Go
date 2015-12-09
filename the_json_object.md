## The JSON object

It is expected that many websocket clients and servers will exchange data in JSON format. For Go programs this means that a Go object will be marshalled into JSON format as described in [Chapter 4: Serialisation](../dataserialisation/README.md) and then sent as a UTF-8 string, while the receiver will read this string and unmarshal it back into a Go object.

The `websocket` convenience object `JSON` will do this for you. It has methods `Send` and `Receive` for sending and receiving data, just like the `Message` object.

A client that sends a `Person` object in JSON format is

```go
/* PersonClientJSON
 */
package main

import (
	"golang.org/x/net/websocket"
	"fmt"
	"os"
)

type Person struct {
	Name   string
	Emails []string
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "ws://host:port")
		os.Exit(1)
	}
	service := os.Args[1]

	conn, err := websocket.Dial(service, "",
		"http://localhost")
	checkError(err)

	person := Person{Name: "Jan",
		Emails: []string{"ja@newmarch.name", "jan.newmarch@gmail.com"},
	}

	err = websocket.JSON.Send(conn, person)
	if err != nil {
		fmt.Println("Couldn't send msg " + err.Error())
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

and a server that reads it is

```go
/* PersonServerJSON
 */
package main

import (
	"golang.org/x/net/websocket"
	"fmt"
	"net/http"
	"os"
)

type Person struct {
	Name   string
	Emails []string
}

func ReceivePerson(ws *websocket.Conn) {
	var person Person
	err := websocket.JSON.Receive(ws, &person)
	if err != nil {
		fmt.Println("Can't receive")
	} else {

		fmt.Println("Name: " + person.Name)
		for _, e := range person.Emails {
			fmt.Println("An email: " + e)
		}
	}
}

func main() {

	http.Handle("/", websocket.Handler(ReceivePerson))
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
