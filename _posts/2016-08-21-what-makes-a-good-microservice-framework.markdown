---
layout: post
title:  "What makes a good microservice framework"
tags: [microservices, thoughts]
author: nic
image: /images/posts/good-frameworks/mindmap.png
---
This is of course an entirely subjective opinion for most people however whilst I have been thinking about this and how it could be possible to score and rank the various frameworks I thought I would share a mind map I have made.

[![MindMap](/images/posts/good-frameworks/mindmap.png)](/images/posts/good-frameworks/mindmap_large.png)


## Ease of use?
The first category I wanted to look at was the Ease of use, any good framework should be easy to use, this category has been broken down into the following sub categories.

### Fast setup
The framework should be fast to setup and with the minimum number of steps and dependencies.

### Code generation
It should support code generation templates to scaffold an initial service and possibly extend the service.

### Tooling
There must be adequate tooling.

- CI/CD: Must integrate and work well with Continuous Integration and Continuous Deployment pipelines.  The tooling must be scriptable from bash.
- Cross platform: The tools must work cross platform, Mac and Linux as a bare minimum.

### Maintainable
The framework must be maintainable with the minimum of effort.

- Easy to update: It must be easy to update with the minimum of code changes.
- Well versioned: The framework must be well versioned with breaking changes to the API mainly restricted to Major version updates.


## Format
Ideally the service will support good standards in one of the following formats.

### REST
If the framework implements REST it must take full advantage of semantic API design with appropriate use of HTTP verbs and Status Codes.

### RPC
If the framework is RPC based it must use a standard and open messaging protocol.


## Patterns
The framework must implement the standard patterns of microservice architecture.

### Circuit breaking
Client calls to downstream services must implement circuit breaking.

### Service discovery
It must be capable of registering with a dynamic service registry and capable of querying the same registry to locate connected services.

- Consul
- Etcd
- DNS SRV records
- Proprietory: Proprietary service registries must be open and usable from other clients which do no implement the framework or its SDKs.

### Timeouts
Downstream client calls must be equipped with a user configurable timeout.

### Health checks
The framework must create an automatic health check endpoint.

### Routing
The framework must support multiple routes with an easy to use pattern based matching.

### Middleware
The framework must support middleware to allow the user to create shared code for handlers.

### Load balancing
Downstream client connections should be capable of load balancing.

## Language independence
The framework needs to be language independent to enable cross team polyglot workflow.

### Polyglot
At a minimum it should be possible to create client SDKs in multiple languages.


## Ability to interface with other frameworks
It must be possible to interact with any service built with the framework by clients who are not built using the same framework.

### Implements standards
A standard message protocol should be used to maximize interaction.

- JSON-RPC
- Thrift
- Protobuffers
- JSON

### Open
The framework should be open in both the source code and the roadmap.


## Extensible
When required the user should be able to extend the framework.

### Plugins
A pluggable software architecture to be able to create generators and templates.

### Middleware
Extension through handler middleware.


## Support
A good support network is incredibly important throughout the lifecycle of the service.

### Maintained
The framework should be well maintained with:

- Regular updates
- Accepts pull requests
- Corporate sponsor: Whilst this option is not essential a corporate sponsor can extend the lifecycle of a framework as there is less likelihood of a  [leftpad] situation.

### Documentation
The framework should be well documented with clear and concise examples and comprehensive API documentation.

- Easy to follow: Documentation should be accessible and easy to read.
- Code samples: Adequate code samples should be provided to support a developer using the framework. 

### Tutorials
The framework will ideally have community contributed tutorials in both blog and video formats.

### Community
There should be a good community using and supporting the framework with at least one of the following channels of communication.

- Slack
- Gitter
- Github
- Mailing list
- Stack Overflow


## Secure
The framework should be secure and implement the latest industry standards.

### TLS
It should be possible to secure the endpoints of the framework using TLS.

### OWASP
The framework should implement OWASP advisory.

### Validation
Requests should be automatically validated based on rules implemented by message annotation.  

### Well patched
Security vulnerabilities should be regularly assessed and patched.


## Open source
The framework should be open sourced and released on a license which allows forking and modification.

### Community
There should be good open source community following and contribution for the project.

### Popular
The framework should be popular and commercially used.


## Quality
The code quality of the service should be visible and of a high standard. Community contributions should follow a published process and standard.

### High test coverage
Test coverage should be high and monitored, pull requests should ensure adherence to coding standards.

- Unit
  High number of fast running unit tests is essential for both the framework and the generated code.
- Behavioural / functional: Ideally the framework should implement behavioral and functional tests in terms of the generated code and the build process. 

### Automated builds
Automated builds of the source code should be present and visible.  Pull requests should run an automated build and the state reported on the request.

### Code quality
Automated code quality tools should be used and the results visible.

- Coveralls
- Code climate
- Sonar

### Standard language patterns
A standard method of writing the code taking account of the language level idioms is essential.


## Efficient
The framework must produce code which is efficient when run.

### Fast
The code must execute quickly and be designed for performance.

### Low latency
Requests should be low latency.

### Low memory
The service should be memory efficient.

### Supports large number of connections
It should support a large number of concurrent connections.
