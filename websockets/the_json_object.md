## The JSON object

It is expected that many websocket clients and servers will exchange data in JSON format. For Go programs this means that a Go object will be marshalled into JSON format as described in [Chapter 4: Serialisation](../dataserialisation/README.md) and then sent as a UTF-8 string, while the receiver will read this string and unmarshal it back into a Go object.

The `websocket` convenience object `JSON` will do this for you. It has methods `Send` and `Receive` for sending and receiving data, just like the `Message` object.

A client that sends a `Person` object in JSON format is

```go
/* PersonClientJSON
 */
package main

import (
	"log"
	"os"

	"golang.org/x/net/websocket"
)

type Person struct {
	Name   string
	Emails []string
}

func main() {
	if len(os.Args) != 2 {
		log.Fatalf("Usage: %v ws://host:port", os.Args[0])
	}
	service := os.Args[1]

	conn, err := websocket.Dial(service, "", "http://localhost")
	checkError(err)

	person := Person{
		Name:   "Jan",
		Emails: []string{"ja@newmarch.name", "jan.newmarch@gmail.com"},
	}

	err = websocket.JSON.Send(conn, person)
	if err != nil {
		log.Fatalf("Couldn't send msg %v", err)
	}
}

func checkError(err error) {
	if err != nil {
		log.Fatalf("Fatal error %v", err)
	}
}
```

and a server that reads it is

```go
/* PersonServerJSON
 */
package main

import (
	"fmt"
	"log"
	"net/http"

	"golang.org/x/net/websocket"
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
		return
	}
	fmt.Println("Name: " + person.Name)
	for _, e := range person.Emails {
		fmt.Println("An email: " + e)
	}
}

func main() {
	http.Handle("/", websocket.Handler(ReceivePerson))
	err := http.ListenAndServe(":12345", nil)
	checkError(err)
}

func checkError(err error) {
	if err != nil {
		log.Fatalf("Fatal error %v", err)
	}
}
```
