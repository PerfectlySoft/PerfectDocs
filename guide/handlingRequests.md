# Handling Requests
As an internet server, Perfect's main function is to receive and respond to requests from clients.  Perfect provides objects to represent the request and the response components, and it permits you to install handlers to control the resulting content generation.Everything begins with the creation of the server object. The server object is configured and subsequently binds and listens for connections on a particular port. When a connection occurs, the server begins reading the request data. Once the request has been completely read, the server will pass the [request object](HTTPRequest.md) through any request filters. 

These filters permit the incoming request to be modified. The server will then use the request's path URI and search the [routing](routing.md) system for an appropriate handler. If a handler is found, it is given a chance to populate a [response object](HTTPResponse.md). Once the handler indicates that it is done responding, the response object is passed through any response filters. These filters permit the outgoing data to be modified. The resulting data is then pushed to the client, and the connection is either closed, or it can be reused as an HTTP persistent connection, aka HTTP keep-alive, for additional requests and responses.

Consult the following sections for more details on each specific phase and what can be accomplished during each:

* [Routing](routing.md) - Describes the routing system and shows how to install URL handlers
* [HTTPRequest](HTTPRequest.md) - Provides details on the request object protocol
* [HTTPResponse](HTTPResponse.md) - Provides details on the response object protocol
* [Request &amp; Response Filters](filters.md) - Shows how to add filters and illustrates how they are useful

In addition, the following sections show how to use some of the pre-made, specialized handlers to accomplish specific tasks:

* [Static File Handler](staticFileContent.md) - Describes how to serve static file content
* [Mustache](mustache.md) - Shows how to populate and serve Mustache template based content
