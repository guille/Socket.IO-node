Socket.IO Server: Sockets for the rest of us
============================================

The `Socket.IO` server provides seamless supports for a variety of transports intended for realtime communication

- WebSocket (with Flash policy support)
- Server-Sent Events
- XHR Polling
- XHR Multipart Streaming
- Forever Iframe

Requirements
------------

- Node v0.1.32+
- [Socket.IO client](http://github.com/RosePad/Socket.IO) to connect from the browser

How to use
----------

`Socket.IO` is designed not to take over an entire port or Node `http.Server` instance. This means that if you choose your HTTP server to listen on the port 80, `socket.io` can intercept requests directed to it and the normal requests will still be served.

By default, the server will intercept requests that contain `socket.io` in the path / resource part of the URI. You can change this (look at the available options below).

	var http = require('http'), 
			io = require('./socket.io/socket.io.js'),
			
	server = http.createServer(function(req, res){
		// your normal server code
		res.writeHeader(200, {'Content-Type': 'text/html'});
		res.writeBody('<h1>Hello world</h1>');
		res.finish();
	});
			
	// socket.io, I choose you
	io.listen(server);
	
Due to a lack of flexibility in the current Node HTTP server implementation, you'll have to patch Node before using `socket.io`.
In the node directory run:
	
	patch -p1 < {../directory/to/socket.io-node}/patch/{node version}.patch
	./configure
	make
	make test
	sudo make install
	
On the client side, you should use the [Socket.IO client](https://github.com/RosePad/Socket.IO) to connect.

## Demo

To run the demo, go to `test` directory and run

	sudo node server.js
	
and point your browser to http://localhost:8080. In addition to 8080, if the transport `flashsocket` is enabled, a server will be initialized to listen to requests on the port 843.

## Documentation

### Listener

	io.listen(<http.Server>, [options])
	
Returns: a `Listener` instance
	
Public Properties:

- *server*

	The instance of _process.http.Server_
	
- *options*

	The passed in options combined with the defaults
	
- *clients*
	
	An array of clients. Important: disconnected clients are set to null, the array is not spliced.
	
- *clientsIndex*

	An object of clients indexed by their session ids.
	
Methods:

- *addListener(event, λ)*

	Adds a listener for the specified event. Optionally, you can pass it as an option to `io.listen`, prefixed by `on`. For example: `onClientConnect: function(){}`
	
- *removeListener(event, λ)*	

	Remove a listener from the listener array for the specified event.
	
- *broadcast(message, [except])*

	Broadcasts a message to all clients. There's an optional second argument which is an array of session ids or a single session id to avoid broadcasting to.
	
Options:
	
- *resource*

		socket.io

  The resource is what allows the `socket.io` server to identify incoming connections by `socket.io` clients. Make sure they're in sync.

- *transports*

		['websocket', 'server-events', 'flashsocket', 'htmlfile', 'xhr-multipart', 'xhr-polling']
		
	A list of the accepted transports.
	
- *timeout*

		8000
		
	Time it has to pass without a reconnection to consider a client disconnected. Applies to all transports.
	
- *log*
	
		ƒ(){ sys.log }
		
	The logging function. Defaults to outputting to stdout through `sys.log`

Events:
	
- *clientConnect(client)*
	
	Fired when a client is connected. Receives the Client instance as parameter
	
- *clientMessage(message, client)*

	Fired when a message from a client is received. Receives the message and Client instance as parameter
	
- *clientDisconnect(client)*

	Fired when a client is disconnected. Receives the Client instance as parameter

Important note: `this` in the event listener refers to the `Listener` instance.

### Client
	
	Client(listener, req, res)
	
Public Properties:

- *listener*

	The `Listener` instance this client belongs to.

- *connected*

	Whether the client is connected
	
- *connections*

	Number of times the client connected
	
Methods:

- *send(message)*

	Sends a message to the client
	
- *broadcast(message)*

	Sends a message to all other clients. Equivalent to Listener::broadcast(message, client.sessionId)

## Protocol

One of the design goals is that you should be able to implement whatever protocol you desire without `Socket.IO` getting in the way. `Socket.IO` has a minimal, unobtrusive protocol layer. It consists of two parts:

* Connection handshake
	
	This is required to simulate a full duplex socket with transports such as XHR Polling or Server-sent Events (which is a "one-way socket"). The basic idea is that the first message received from the server will be a JSON object that contains a session id that will be used for further communication exchanged between the client and the server. 
	
	The concept of session also benefits naturally full-duplex WebSocket, in the event of an accidental disconnection and a quick reconnection. Messages that the server intends to deliver to the client are cached temporarily until the reconnection.
	
	The implementation of reconnection logic (potentially with retries) is left for the user.
	
* Message batching

	In order to optimize the resources, messages are batched. In the event of the server trying to send multiple messages while the client is temporarily disconnected (ie: xhr polling), and messages are stacked, or messages being stacked prior to the handshake being successful, a JSON object containing a list (array) of messages is sent to the client.

Despite this extra layer, your messages are delivered unaltered to the different event listeners. You can JSON.stringify() objects, send XML, or maybe plain text.

## Credits

Guillermo Rauch [guillermo@rosepad.com]

Special thanks to [Jonas Pfenniger](http://github.com/zimbatm) for his workaround patch to keep the HTTPConnection open after the request is successful.

## License 

(The MIT License)

Copyright (c) 2009 RosePad &lt;dev@rosepad.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.