# An HTTP & Web Services Primer

Most of the web as we know it, and as applies to the common usage of server side development, is built upon only a few main technologies.

This document will attempt to provide an overview of some of these as they relate to Perfect, Server Side Swift, and general API development.

## What is an API?

The acronym "API" stands for "Application Programming Interface". Put more simply, an API is a bridge between two systems. It generally accepts a standardized type and style of input, transforms it internally and works with another system to perform an action or return information.

An example of an API is GitHub's API: Most will be familiar with using GitHub's web interface, but they also have an API. This allows applications such as task and issue management systems like [Jira](https://www.atlassian.com/software/jira) and continuous integration systems (CI) like [Bamboo](https://www.atlassian.com/software/bamboo) to link themselves directly to your repositories to **provide greater functionality than any one system could by itself.**

An API like this is generally built upon HTTP or HTTPS.

## HTTP, HTTPS

"HTTP" is the common acronym for "HyperText Transfer Protocol" - it is is a set of standards that allow users of the World Wide Web to exchange information found on web pages.

"HTTPS" is a protocol for secure HTTP communication. Each end of the communication agree on a trusted "key" that encrypts the information being trasnmitted, with the ability to decrypt it on receipt. The more complex the security, the harder it is for a third patry to intercept and read it - and potentially change it.

When you access a web site in your browser, it will likely be over HTTP, or if the "lock" shows it will be using HTTPS.

When an iOS or Android application accesses a backend service to get information such as the current weather - it is using HTTP or more likely, HTTPS.

## The general form of an API

### Routes

An API consists of "routes", which are similar to directory/folder name paths through a file system to a document. Each route points to a different action you wish to perform. A "show me a list of users" route is different to a "create a new user" route.

In an API these routes will often not exist as actual directories and documents, but as pointers to functions. The "directory structure" implied is usually a way of logically grouping functionality - for example:

```
/api/v1/users/list
/api/v1/users/detail
/api/v1/users/create
/api/v1/users/modify
/api/v1/users/delete

/api/v1/companies/list
/api/v1/companies/detail
/api/v1/companies/create
/api/v1/companies/modify
/api/v1/companies/delete
```

This example illustrates a typical "CRUD" system. "CRUD" means "Create, Read, Update, Delete". The difference between the two groups seems to only be the `users` vs. `companies` part but they will point to different functions.

Additionally, the "detail", "modify" and "delete" routes will include a unique identifier for the record. For example:

```
/api/v1/users/list
/api/v1/users/detail?id=5d096846-a000-43db-b6c5-a5883135d71d
/api/v1/users/create
/api/v1/users/modify?id=5d096846-a000-43db-b6c5-a5883135d71d
/api/v1/users/delete?id=5d096846-a000-43db-b6c5-a5883135d71d
```

This example shows these routes passing to the server an id parameter that relates to a very specific record. The list and create do now have an id because an id for those routes is irrelevant.

In addition to routes, a request to every one of these individual routes will include an HTTP "Verb".

### HTTP Verbs

HTTP Verbs are an additional piece of information that a web browser or mobile application client supplies with every request to a route. These verbs can give additional "context" to the API Server as to the nature of the data being received.

Common HTTP Verbs:

#### GET 
The GET method requests a specified route, and the only parameters passed from client to server are in the URL. Requests using GET should only retrieve data and should have no other effect. i.e. using a GET request to delete a database record is possible but not recommended.

#### POST 
Normally used for sending info to create a new record in a database, a POST request is what normally gets submited when you fill in a form on a website. The name-value pairs of the data are submitted in a POST request's "POST Body" and are read by the API server as discrete pairs.

#### PATCH
PATCH requests are generally considered to be the same as a POST but used for updates.

#### PUT 
Used predominantly for file uploads, a PUT request will include a file in the body of the request sent to the server.

#### DELETE 
A DELETE request is the most descriptive of all the HTTP verbs. When sending a DELETE request you are instructing the API server to remove a specific resource. Usually some form of uniquely identifying information is included in the URL.


### Using HTTP Verbs and Routes together to simplify an API

When used together these two components of a request can reduce the perceived complexity of an API.

Looking at the following structure with the HTTP verb followed by a URL, you will see that the process is much simpler and more specific.


```
GET     /api/v1/users
GET     /api/v1/users/5d096846-a000-43db-b6c5-a5883135d71d
POST    /api/v1/users
PATCH   /api/v1/users/5d096846-a000-43db-b6c5-a5883135d71d
DELETE  /api/v1/users/5d096846-a000-43db-b6c5-a5883135d71d
```

At first glance all seem to be pointing to the same route - `/api/v1/users`, however each route actually performs a different action. Similarly the difference between the first two GET routes is the id appended as a URL compnent, not a parameter as shown inthe earlier example. This is usually achieved with a "wildcard" specified in the route setup.

The next step is to review the "Handling Requests" chapters in the Perfect documentation. These sections will outline how to implement routes pointing to functions and methods, how to access information passed to the API server from a frontend, whether it be a web browser or a mobile application.