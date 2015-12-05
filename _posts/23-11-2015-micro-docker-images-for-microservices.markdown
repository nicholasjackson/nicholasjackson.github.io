---
layout: post
title:  "Micro Docker Images for Go Microservices"
date:   2015-12-01 14:00:00
categories: other
comments: true
---
Unless you have been living in a cave on a deserted island in the middle of the pacific which has been wrapped in a faraday cage then you are probably talking about microservices and Docker.  This is by far the best topic to get you a pay rise since you pretended to know what Agile is.  In truth however microservices are really important for the way we work, they allow us to develop with independence, keep code contained by its function and if we are using Docker or another container technology, deploy exactly the same application we have just been running on our dev machines.

When we first get into building Docker images the common approach appears to be to follow the way we used to build virtual machines which is to use Ubuntu, Debian or another big name distribution.  The problem with this is that it produces images that are not that micro, the base image of Ubuntu alone is 187.9MB and most of this is software and libraries which we will never use.

![Container Architecture]({{ site.url }}/images/post_images/micro_containers/containers_vs_vms.png)  

The main difference between a Container and a VM is that a container runs in process isolation on top of the core OS therefore it does not need its own kernel and a billion other files to run.  What it really needs are executables and libraries and this is the important part of a microcontainer, only install what you are intending to use, do we really need Python for our Go microservice? This is not a difficult problem to solve and let me explain how:

## Micro base box
Firstly let's ditch the goto Ubuntu or Debian image you are using as your base box and take a look at something a little lighter.

Alpine Linux ([http://www.alpinelinux.org/](http://www.alpinelinux.org/)) is a small linux distribution which when used as a base container results in roughly a 5 MB image size, it is based on the BusyBox ([http://www.busybox.net/about.html](http://www.busybox.net/about.html)) distribution of linux which was originally designed for embedded systems.  So why not just use BusyBox? There are quite a few advantages to using Alpine the first is convenience in that Alpine has a package manager `apk` which can simplify some of your container maintenance and the second is that the kernel has been patched with PaX ([https://www.grsecurity.net/](https://www.grsecurity.net/)) which protects against quite a large number of 0 day vulnerabilities.

You can see from the table below, all this only adds 4.1 MB to the size of busybox but for this convenience I think is well worth the cost.

{% highlight bash %}
REPOSITORY                             VIRTUAL SIZE
ubuntu                                 187.90 MB  
nicholasjackson/microservice-basebox    15.82 MB  
alpine                                   5.24 MB  
busybox                                  1.10 MB
{% endhighlight %}

## Compiling Go applications for Alpine Linux
One of the advantages of working with Go for your microservices is the way that it creates compiled binaries which require no framework or runtime dependancies, this works really nicely with Alpine however we do need to remember that since it is a light weight distribution it does not have all the C libraries which would be normally expected for a standard dynamically linked Go binary.  Fortunately there is a simple work around which is to disable cgo when we compile and statically link the application.  We also need to tell the compiler to rebuild all the packages we are using to ensure they are also rebuilt and statically linked.

{% highlight bash %}
$ CGO_ENABLED=0 go build -a -installsuffix cgo .
{% endhighlight %}

Lets take a look at this in a little more detail:  
`CGO_ENABLED=0` is a flag to tell the build system to ignore cgo and to statically link the binary  
`-a` is the flag to force rebuilding of packages that are already up-to-date.  
`-installsuffix cgo` adds a suffix to use in the name of the package installation directory, this keeps output separate from default builds variants.  

## Running your application
That gives us our binary what about your config, if you are building a microservice architecture then you are most likely going to be using Consul ([https://www.consul.io/](https://www.consul.io/)), if not you should really check it out as it is amazing.  For the purpose of this article I am going to assume that you are and therefore you are also using Consul template ([https://github.com/hashicorp/consul-template](https://github.com/hashicorp/consul-template)) to populate your config file and possibly restart your application when the config changes, so we will need that too.

But we need to keep both of those things running on the container for this we would usually start it with a daemon runner like Supervisor ([http://supervisord.org/](http://supervisord.org/)).  Unfortunately this uses Python which we do not want on our light weight linux box so instead we will use Skaware S6 ([http://skarnet.org/software/s6/](http://skarnet.org/software/s6/)).

S6 is a simple and effective application, it is started using the s6-svscan application with the parameter of a config folder `$ s6-svscan /etc/s6`.  If you look at the example config folder in my basebox ([https://hub.docker.com/r/nicholasjackson/microservice-basebox/](https://hub.docker.com/r/nicholasjackson/microservice-basebox/)) repository you will see that it contains multiple folders containing a bunch of shell scripts.

### .s6-svscan
This folder contains two scripts "crash" and "finish", crash is called when one of the applications s6 is running terminate with an error, finish is called when the s6 process shuts down.

### app
This folder contains two scripts "run" and "finish", run is the script to start your main service and will typically be the execution path for your go binary.  Finish is where you can put any activities to run when the application shuts down.

### consul-template
This folder also contains the same two scripts but this time it is for the consul-template application, consul-template generates the config file used by our microservice from information stored in the Consul server.  Since this continually monitors the information in Consul this application must also be running for the life cycle of our microservice so we need to use s6 to monitor it and keep it alive.

## Docker Base Image
For convenience I have created a base Docker image based on Alpine with Skaware S6 and Consul Template installed ([https://hub.docker.com/r/nicholasjackson/microservice-basebox/](https://hub.docker.com/r/nicholasjackson/microservice-basebox/)), all in the new base box will be a tiny 15.82 MB.  By the time you add your bloated application code (yes I am being facetious) the total image will be roughly 30 MB which is a staggering 13% of the size should you have chosen the Ubuntu base box.

## Conclusion
In this article I have mainly discussed Go applications but you can apply the techniques to Ruby or Python.  Alpine "apk" already has packages for both of these languages and with a bit of effort you could probably get a Java JRE on there.
