## The Codec type

The `Message` and `JSON` objects are both instances of the type `Codec`. This type is defined by

```go
type Codec struct {
    Marshal   func(v interface{}) (data []byte, payloadType byte, err error)
    Unmarshal func(data []byte, payloadType byte, v interface{}) (err error)
}
```

The type `Codec` implements the `Send` and `Receive` methods used earlier.

It is likely that websockets will also be used to exchange XML data. We can build an XML `Codec` object by wrapping the XML marshal and unmarshal methods discussed in [Chapter 12: XML](../xml/README.md) to give a suitable `Codec` object.

We can create a `XMLCodec` package in this way:

```go
package xmlcodec

import (
	"encoding/xml"
	"golang.org/x/net/websocket"
)

func xmlMarshal(v interface{}) (msg []byte, payloadType byte, err error) {
	//buff := &bytes.Buffer{}
	msg, err = xml.Marshal(v)
	//msgRet := buff.Bytes()
	return msg, websocket.TextFrame, nil
}

func xmlUnmarshal(msg []byte, payloadType byte, v interface{}) (err error) {
	// r := bytes.NewBuffer(msg)
	err = xml.Unmarshal(msg, v)
	return err
}

var XMLCodec = websocket.Codec{xmlMarshal, xmlUnmarshal}
```

We can then serialise Go objects such as a `Person` into an XML document and send it from a client to a server by

```go
/* PersonClientXML
 */
package main

import (
	"golang.org/x/net/websocket"
	"fmt"
	"os"
	"xmlcodec"
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

	conn, err := websocket.Dial(service, "", "http://localhost")
	checkError(err)

	person := Person{Name: "Jan",
		Emails: []string{"ja@newmarch.name", "jan.newmarch@gmail.com"},
	}

	err = xmlcodec.XMLCodec.Send(conn, person)
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

A server which receives this and just prints information to the console is

```go
/* PersonServerXML
 */
package main

import (
	"golang.org/x/net/websocket"
	"fmt"
	"net/http"
	"os"
	"xmlcodec"
)

type Person struct {
	Name   string
	Emails []string
}

func ReceivePerson(ws *websocket.Conn) {
	var person Person
	err := xmlcodec.XMLCodec.Receive(ws, &person)
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

