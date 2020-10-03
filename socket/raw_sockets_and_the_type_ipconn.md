## Raw sockets and the type IPConn

This section covers advanced material which most programmers are unlikely to need. It deals with *raw sockets*, which allow the programmer to build their own IP protocols, or use protocols other than TCP or UDP

TCP and UDP are not the only protocols built above the IP layer. The [site]( http://www.iana.org/assignments/protocol-numbers) lists about 140 of them (this list is often available on Unix systems in the file `/etc/protocols`). TCP and UDP are only numbers 6 and 17 respectively on this list.

Go allows you to build so-called raw sockets, to enable you to communicate using one of these other protocols, or even to build your own. 
But it gives minimal support: it will connect hosts, and write and read packets between the hosts. 
In the next chapter we will look at designing and implementing your own protocols above TCP; this section considers the same type of problem, but at the IP layer.

To keep things simple, we shall use almost the simplest possible example: how to send a ping message to a host. Ping uses the "echo" command from the *ICMP* protocol. This is a byte-oriented protocol, in which the client sends a stream of bytes to another host, and the host replies. the format is:

* The first byte is 8, standing for the echo message
* The second byte is zero
* The third and fourth bytes are a checksum on the entire message
* The fifth and sixth bytes are an arbitrary identifier
* The seventh and eight bytes are an arbitrary sequence number
* The rest of the packet is user data


The following program will prepare an IP connection, send a ping request to a host and get a reply. You may need to have root access in order to run it successfully.

```go
/* Ping
 */
package main

import (
	"bytes"
	"fmt"
	"io"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "host")
		os.Exit(1)
	}

	addr, err := net.ResolveIPAddr("ip", os.Args[1])
	if err != nil {
		fmt.Println("Resolution error", err.Error())
		os.Exit(1)
	}

	conn, err := net.DialIP("ip4:icmp", addr, addr)
	checkError(err)

	var msg [512]byte
	msg[0] = 8  // echo
	msg[1] = 0  // code 0
	msg[2] = 0  // checksum, fix later
	msg[3] = 0  // checksum, fix later
	msg[4] = 0  // identifier[0]
	msg[5] = 13 //identifier[1]
	msg[6] = 0  // sequence[0]
	msg[7] = 37 // sequence[1]
	len := 8

	check := checkSum(msg[0:len])
	msg[2] = byte(check >> 8)
	msg[3] = byte(check & 255)

	_, err = conn.Write(msg[0:len])
	checkError(err)

	_, err = conn.Read(msg[0:])
	checkError(err)

	fmt.Println("Got response")
	if msg[5] == 13 {
		fmt.Println("identifier matches")
	}
	if msg[7] == 37 {
		fmt.Println("Sequence matches")
	}

	os.Exit(0)
}

func checkSum(msg []byte) uint16 {
	sum := 0

	// assume even for now
	for n := 1; n < len(msg)-1; n += 2 {
		sum += int(msg[n])*256 + int(msg[n+1])
	}
	sum = (sum >> 16) + (sum & 0xffff)
	sum += (sum >> 16)
	var answer uint16 = uint16(^sum)
	return answer
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}
```

## Conclusion

This chapter has considered programming at the IP, TCP and UDP levels. 
This is often necessary if you wish to implement your own protocol, or build a client or server for an existing protocol. 
