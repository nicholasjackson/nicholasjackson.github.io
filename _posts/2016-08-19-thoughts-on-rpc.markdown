---
layout: post
title:  "My struggles with RPC"
tags: [microservices, rpc, thoughts]
author: nic
image: /images/posts/rpc/grpc_concept.png
---
I have decided to write up my thinking on some investigation I have done with gRPC and protobuffers. At first I hated the concept as I was fully entrenched in the "Why use anything other than REST" camp, however from playing round a little with gRPC I am starting to feel that RPC with a DSL for defining contracts with protobuffers could be the way to go.

My argument for REST has always been the accessibility of it, however gRPC supports 10 languages from the official packages and many more from community packages.  The benefits of protobuffers are that you can generate a client in your chosen language using the protoc application, this allows DRYness of code across the many consumers of your API.  Sure you can use Swagger to define your API and then code gen from the Swagger files but this process defeats the purpose of defining an API in a RESTful way will all the semantics of HTTP verbs to infer intent.  If you are generating clients then why not take advantage of the efficiency of RPC based communication and also allow yourself the benefits of using light weight binary messaging.



## Versioning:
No more complex than when working with REST, use semantic versioning can be implemented in the same way as you can in rest, gRPC is not like JMI where the exact same interface must be implemented on both the client and the server, you can add elements to the server side interface and by implementing the same standard rules you would follow with a RESTful API you are in a no worse position.

### Breaking changes (update Major version, client contract update required):
* Removing or renaming APIs or API parameters;
* Changing the type of an API parameter for example from Integer to String;
* Changes to response codes, error codes or fault contracts;
* Changes to the behavior of an existing API.

### Things that do not involve a breaking change (No client updated required unless desired):
* Adding parameters to a returned entity;
* Adding additional endpoints or functionality;
* Bug fixes or other maintenance which do not included items in the breaking changes list.


## Web access:
This is where RPC falls down however there is an architectural pattern called a HTML → RPC reverse proxy which is explained a little bit in this blog post: (http://www.grpc.io/blog/coreos)[http://www.grpc.io/blog/coreos]. Whilst this can seem like a problem it is restricted to your Public APIs.  Following the reverse proxy pattern you can easily create RESTful endpoints for browser based clients and support RPC for native apps.  Internal services can consume the RPC client with no problems.


## Documentation
Your proto files are the documentation, creating comments inside the file is all that is really needed, 

```proto
// The greeting service definition.
service Users {
  // Create user creates a user in the system with the given User details,
  // it returns an Error message which will be nil on a successful operation
  rpc CreateUser (User) returns (Error) {}
}

// Person describes a user entity
message User {
  // name is a required field and represents the name of
  required string name = 1;
  // id is the unique identifier for the user in the system
  required int32 id = 2;
  // email is the users email address and is an optional field 
  optional string email = 3;
}

message Error {
  optional code int32 = 1
  optional detail string = 2
}

```


## Advantages / Disadvantages

### Advantages
* Ability to easily implement one / two way streaming for data;
* Auto generated client code;
* Fast and low latency communication;
* Native mobile clients get a huge boost by avoiding HTTP;
* Fantastic open source support with gRPC, Thrift, JSON-RPC.

### Disadvantages
* Inability to leverage HTTP caching provided by a CDN;
* Possibility of having to implement a reverse proxy to support browser based clients;
* Testing, you can't just CURL an endpoint and it may be required to build a test harness.


## Conclusion
I will certainly be investigating gRPC and protobuffers further and can see myself using this as the defacto communication standard for my microservices.  


## Further reading
* gRPC Framework (http://www.grpc.io)[http://www.grpc.io]
* Thrift Framework (https://thrift.apache.org)[https://thrift.apache.org]
* JSON-RPC Standard (http://json-rpc.org)[http://json-rpc.org]