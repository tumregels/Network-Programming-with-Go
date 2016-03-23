## Message Format

In the last chapter we discussed some possibilities for representing data to be sent across the wire. Now we look one level up, to the messages which may contain such data.

* The client and server will exchange messages with different meanings, e.g.
    * Login request,
    * get record request,
    * login reply,
    * record data reply. 
* The client will prepare a request which must be understood by  the server.
* The server will prepare a reply which must be understood by the client. 

Commonly, the first part of the message will be a message type.

* Client to server

        LOGIN name passwd
        GET cpe4001 grade
          

* Server to client

        LOGIN succeeded
        GRADE cpe4001 D
          

The message types can be strings or integers. e.g. HTTP uses integers such as 404 to mean "not found" (although these integers are written as strings). The messages from client to server and vice versa are disjoint: "LOGIN" from client to server is different to "LOGIN" from server to client. 
