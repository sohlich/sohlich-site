+++
author = ""
categories = []
description = ""
draft = true
linktitle = ""
featured = ""
featuredpath = ""
featuredalt = ""
date = "2016-12-03T14:16:15+01:00"
title = "Metro - SSH tunneling tool"

+++

# Metro - SSH tunneling tool

One of the most powerful sides of the golang, in my opinion, is that
it let you use the lower level things that you usually hasitate to
touch. The lower level things I mean the protocols, 
access to system sources and things like these.

The reasons should be various: the area is too complex for you ,
the libraries for the programming language is some kind of cumbersome or 
just it is not possible. 

For me, one of those was an SSH protocol and the stuff around. 
I'm not very well grounded person in these things, so for me it was like the magic 
that is usually done on the bash level 
or only C/C++ devs are the right ones to touch this.
As a Java developer, I'm used to work with libs 
like JSch but the feeling with that is somehow strange.

I decided to discover how it is done by golang. 
The blog post [SSH tunneling in Golang](http://blog.ralch.com/tutorial/golang-ssh-tunneling/) clearly 
describes how to handle this and actually inspires me to do something useful with it.

My daily job is now full of SSH tunelling. We are 
Microsoft products based company so commonly We use Putty as a standart tool to do all the tunneling stuff. 
I was little bored with the UI and I was decided to implement 
simple tunelling tool with file based configuration by myself with help of golang.

This is how the "Metro" was born. The name shoudl symbolize the real metro with all the tunnels.
The implementation itself uses the "golang.org/x/crypto/ssh" library for the 
ssh connection and the rest uses built in libraries to get it work.

The repository link: [github.com/sohlich/metro](https://github.com/sohlich/metro).

