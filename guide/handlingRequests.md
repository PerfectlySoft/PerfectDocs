# Handling Requests
As an internet server, Perfect's main function is to receive and respond to requests from clients. Perfect provides objects to represent the request and the response components and permits you to install handlers to control the resulting content generation.

Everything begins with the creation of the server object. The server object is configured and then binds and listens for connections on a particular port. When a connection occurs the server begins reading the request data.

