## Data Format

There are two main format choices for messages: byte encoded or character encoded.

### Byte format

In the byte format

* the first part of the message is typically a byte to distinguish between message types.
* The message handler would examine this first byte to distinguish message type and then perform a switch to select the appropriate handler for that type.
* Further bytes in the message would contain message content according to a pre-defined format (as discussed in the previous chapter).

The advantages are compactness and hence speed. The disadvantages are caused by the opaqueness of the data: it may be harder to spot errors, harder to debug, require special purpose decoding functions. There are many examples of byte-encoded formats, including major protocols such as DNS and NFS , upto recent ones such as Skype. Of course, if your protocol is not publicly specified, then a byte format can also make it harder for others to reverse-engineer it!

Pseudocode for a byte-format server is

    handleClient(conn) {
        while (true) {
            byte b = conn.readByte()
            switch (b) {
                case MSG_1: ...
                case MSG_2: ...
                ...
            }
        }
    }

Go has basic support for managing byte streams. The interface `Conn` has methods

```go
func (c Conn) Read(b []byte) (n int, err os.Error)
func (c Conn) Write(b []byte) (n int, err os.Error)
```


and these methods are implemented by `TCPConn` and `UDPConn`. 

### Character Format

In this mode, everything is sent as characters if possible. For example, an integer 234 would be sent as, say, the three characters '2', '3' and '4' instead of the one byte 234. Data that is inherently binary may be base64 encoded to change it into a 7-bit format and then sent as ASCII characters, as discussed in the previous chapter.

In character format,

* A message is a sequence of one or more lines
The start of the first line of the message is typically a word that represents the message type.
* String handling functions may be used to decode the message type and data.
* The rest of the first line and successive lines contain the data.
* Line-oriented functions and line-oriented conventions are used to manage this.

Pseudocode is

    handleClient() {
        line = conn.readLine()
        if (line.startsWith(...) {
            ...
        } else if (line.startsWith(...) {
            ...
        }
    }

Character formats are easier to set up and easier to debug. For example, you can use *telnet* to connect to a server on any port, and send client requests to that server. It isn't so easy the other way, but you can use tools like *tcpdump* to snoop on TCP traffic and see immediately what clients are sending to servers.

There is not the same level of support in Go for managing character streams. There are significant issues with character sets and character encodings, and we will explore these issues in a later chapter.

If we just pretend everything is ASCII, like it was once upon a time, then character formats are quite straightforward to deal with. The principal complication at this level is the varying status of *"newline"* across different operating systems. Unix uses the single character `"\n"`. Windows and others (more correctly) use the pair `"\r\n"`. On the internet, the pair `"\r\n"` is most common - Unix systems just need to take care that they don't assume `"\n"`. 
