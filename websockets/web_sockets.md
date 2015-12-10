# Chapter 15 Web sockets

Web sockets are designed to answer a common problem with web systems: the server is unable to initiate or push content to a user agent such as a browser. Web sockets allow a full duplex connection to be established to allow this. Go has nearly complete support for them.

## Warning

The Web Sockets package is not currently in the <a href="https://github.com/golang/go">main Go 1.x tree</a> and is not included in the current distributions. To use it, you need to install it by 

```
go get golang.org/x/net/websocket
```

## Introduction

(Note: The websockets model changed in release r61 of Go 1.0. This describes the current package.)

The standard model of interaction between a web user agent such as a browser and a web server such as Apache is that the user agent makes HTTP requests and the server makes a single reply to each one. In the case of a browser, the request is made by clicking on a link, entering a URL into the address bar, clicking on the forward or back buttons, etc. The response is treated as a new page and is loaded into a browser window.

This traditional model has many drawbacks. The first is that each request opens and closes a new TCP connection. HTTP 1.1 solved this by allowing persistent connections, so that a connection could be held open for a short period to allow for multiple requests (e.g. for images) to be made on the same server.

While HTTP 1.1 persistent connections alleviate the problem of slow loading of a page with many graphics, it does not improve the interaction model. Even with forms, the model is still that of submitting the form and displaying the response as a new page. JavaScript helps in allowing error checking to be performed on form data before submission, but does not change the model.

AJAX (Asynchronous JavaScript and XML) made a significant advance to the user interaction model. This allows a browser to make a request and just use the response to update the display in place using the HTML Document Object Model (DOM). But again the interaction model is the same. AJAX just affects how the browser manages the returned pages. There is no explicit extra support in Go for AJAX, as none is needed: the HTTP server just sees an ordinary HTTP POST request with possibly some XML or JSON data, and this can be dealt with using techniques already discussed.

All of these are still browser to server communication. What is missing is server initiated communications to the browser. This can be filled by Web sockets: the browser (or any user agent) keeps open a long-lived TCP connection to a Web sockets server. The TCP connection allows either side to send arbitrary packets, so any application protocol can be used on a web socket.

How a websocket is started is by the user agent sending a special HTTP request that says "switch to web sockets". The TCP connection underlying the HTTP request is kept open, but both user agent and server switch to using the web sockets protocol instead of getting an HTTP response and closing the socket.

Note that it is still the browser or user agent that initiates the Web socket connection. The browser does not run a TCP server of its own. While the specification is [complex](http://tools.ietf.org/html/draft-ietf-hybi-thewebsocketprotocol-17), the protocol is designed to be fairly easy to use. The client opens an HTTP connection and then replaces the HTTP protocol with its own WS protocol, re-using the same TCP connection.

