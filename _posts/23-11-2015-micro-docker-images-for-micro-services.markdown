---
layout: post
title:  "Micro Docker Images for Micro Services"
date:   2015-02-18 14:53:00
categories: other
comments: true
---
Unless you have been living in a cave on a deserted island in the middle of the pacific which has been wrapped in a faraday cage then you are probably talking about microservices and Docker.  This is by far the latest topic to get you a pay rise since we pretended to know what Agile is.

In truth however microservices are really important for the way we work...

Docker images that are not that Micro

Images based on ubuntu are positively huge

This is fine if you have unlimited bandwidth but we don't, and we sill need to be able to stream Spotify whilst we work.. Seriously tho this is not a difficult problem to solve and let me explain how.

Firstly lets ditch the go-to Ubuntu or Debian image you are using on your base box, and take a look at something a little lighter.

Alpine Linux (http://www.alpinelinux.org/) is a small linux distribution which when used as a base container results in roughly a 5 MB image size it is based on the BusyBox (http://www.busybox.net/about.html) distribution of linux which was originally designed for embedded systems.  So why not just use BusyBox? There are quite a few advantages to using Alpine the first is convenience in that Alpine has a package manager `apk` which can simplify some of your container maintenance and the second is that the kernel has been patched with PaX which protects against quite a large amount of 0 day vulnerabilities.

You can see from the table below, all this only adds 3.9MB to the size of busybox but for this convenience I think is well worth the cost.

```
REPOSITORY                             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                                 latest              e9ae3c220b23        2 weeks ago         187.9 MB
nicholasjackson/microservice-basebox   latest              878785337cbb        6 days ago          15.82 MB
alpine                                 latest              8a648f689ddb        10 weeks ago        5.249 MB
busybox                                latest              c51f86c28340        4 weeks ago         1.109 MB
```

One of the advantages of working with Go for your microservices is the way that it creates compiled binaries which require no framework or runtime dependancies, this works really nicely with Alpine however we do need to remember that since it is a light weight distribution it does not have all the C libraries which would be normally expected for a standard dynamically linked Go binary.  Fortunately there is a simple work around which is to disable cgo which gives us a statically compiled binary.  We also need to tell the compiler to rebuild all the packages we are using to ensure they are also rebuilt and statically linked.

```
CGO_ENABLED=0 go build -a -installsuffix cgo .
```

Lets take a look at this in a little more detail:
`CGO_ENABLED=0` is a flag to tell the build system to ignore cgo and to statically link the binary
`-a` is the flag to force rebuilding of packages that are already up-to-date.
`-installsuffix cgo` a suffix to use in the name of the package installation directory, in order to keep output separate from default builds.

That gives us our binary but we need to keep it running on the container for this we would usually start it with a daemon runner like Supervisor (http://supervisord.org/).  Unfortunately this uses Python which we do not want on our light weight linux box so instead we will use Skaware S6 (http://skarnet.org/software/s6/).

What about your config, well if you are building a Microservice architecture then you are most likely going to be using Consul (https://www.consul.io/), if not you should really check it out as it is amazing.  For the purpose of this article I am going to assume that you are and therefore you are also using Consul template (https://github.com/hashicorp/consul-template) to populate your config file and possibly restart your application when the config changes, so we will need that too.

For convenience I have created a base Docker image based on Alpine with Skaware S6 and Consul Template installed (https://hub.docker.com/r/nicholasjackson/microservice-basebox/), All in all the new base box will be a tiny 15.82 MB.  By the time you add your bloated application code (yes I am being faceicious) the total image will be roughly 30 MB which is a staggering 13% of the size should you have chosen the Ubuntu base box.
