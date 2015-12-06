---
layout: post
title:  "Microservices with Swift"
date:   2015-12-05 12:00:00
categories: swift
comments: true
---
In this post I would like to take a very quick look at the possibilities for using the Swift language to create microservices, initially my intention was create a working example however from some initial investigation I am not convinced that at the present time this would be particularly useful.

![Swift Logo]({{ site.url }}/images/post_images/swift-microservices/swift.png)

## What is Swift
Swift is a language which was initially developed by Apple to replace Objective C for building both iOS and Mac OSX applications ([https://swift.org/about/](https://swift.org/about/)),  on December the 3rd 2015 Apple open sourced Swift ([https://github.com/apple/swift](https://github.com/apple/swift)) and have released packages for both OSX and Ubuntu containing the compiler, debugger and various other tools.  This is an incredible advancement as besides being modern language and in general a pleasure to program it opens up the opportunity for developers to write APIs and cross platform applications with limited change to the codebase.

## Package manager
Essential for a good language are a compliment of good packages, we want to get on with the modeling the business function not developing   libraries for things like encryption.  The swift package manager is a nice idea, defining a Package.swift file in the root of your application you can specify module dependencies from their git locations.  You can also specify the version which  is derived from the tag in the git repository. Again still in its early stages but the roadmap has many features including the ability to deal with packages for different architectures.

## Testing
Testing is as you would with a standard iOS or Mac OSX app, XCTest is fully supported and there are plans to integrate tests into the package building process like Ruby Gems.

## Cross platform
I am pretty excited about the prospect for using Swift cross platform, being able to build microservices is one area but the thought that one day you will also be able to build Swift code for Android is the holy grail for application developers, whilst there are a few projects which are attempting this, Go included, the general stumbling block in the past has been Apple's hesitance to play nice with anything other than their own tools.

The main problem with Swift at the moment for cross platform use is the lack of a complete Foundation layer.  Whilst this is in development and the aim is to have a full implementation as a standard library by version 3 there are quite s few key elements missing which if you are not aware of will mean some code will not run on UNIX as it does not have the full library. [https://github.com/apple/swift-corelibs-foundation/blob/master/Docs/Status.md](https://github.com/apple/swift-corelibs-foundation/blob/master/Docs/Status.md) shows the current status of this project.  NSURLSession and related classes are not yet implemented and this is pretty core to networking, also NSJSONSerialization is currently missing so you might have trouble dealing with the response from a call even if you could make it.

From a personal perspective one thing I dislike about cross platform code is having to think about cross platform code.  Even in the example from Apple the blight of compiler macros are there as you need to use a different random() function for Darwin than you do for UNIX.  I hope the Foundation project will fill most of this void but if not then if will be left to developers to hopefully think cross platform when creating their packages.

## Webserver
In order to adopt Swift for microservices I would want battle tested HTTP server, this is generally the core of most services which implement a RESTFull interface.  Whilst by no means the most trivial problem nor the most complex, a server from a trusted source that I am sure has been patched and tested against vulnerabilities is essential.  It would be nice if there was something in the Foundation framework which dealt with this, like Go's net/http package or an external framework like Node's Express

## Docker
A mandate for microservices these days is can the code run in a container, I am pleased to say that thanks to the Ubuntu packages it is more than possible to create a build container in swift ([https://github.com/swiftdocker/docker-swift](https://github.com/swiftdocker/docker-swift)), this is useful for creating a consistent build environment and for integration into a CI pipeline. When running the application, Swift compiles down to native machine code it should therefore be possible to  have a light weight container based on Alpine or Scratch

## Conclusion
I am pretty excited about the prospects of using Swift for microservice development however I would  most likely wait until version 3 is released and the Foundation package has been completed.  It will also take a little while for the community to get behind the project and start to build out packages for things like Redis, Mongo and MySQL.  At present it would be possible but I personally feel there would be very little gain, the beauty of microservice architecture is that you are not constrained to one language so there is no harm waiting a few months for Swift 3.0 to ship.
