+++
categories = [
]
date = "2017-09-18T18:32:54+02:00"
title = "GO: Don’t afraid of makefiles"
draft = true
thumbnail = ""
tags = [
    "golang","make","development"
]

+++

The golang development provides me the escape from my day job and makes the coding fun again. But even in Go, which has all the awesome tools bundled, the repeated tasks like build and test commands are better to script. The makefile is the way out. There you can keep all your common tasks in one file. Let's go through some useful parts of the makefile. The following listing is complete makefile ready to be used in any golang project.

```
    # Go parameters
    GOCMD=go
    GOBUILD=$(GOCMD) build
    GOCLEAN=$(GOCMD) clean
    GOTEST=$(GOCMD) test
    GOGET=$(GOCMD) get
    BINARY_NAME=mybinary
    BINARY_UNIX=$(BINARY_NAME)_unix
    
    all: test build
    build: 
            $(GOBUILD) -o $(BINARY_NAME) -v
    test: 
            $(GOTEST) -v ./...
    clean: 
            $(GOCLEAN)
            rm -f $(BINARY_NAME)
            rm -f $(BINARY_UNIX)
    run:
            $(GOBUILD) -o $(BINARY_NAME) -v ./...
            ./$(BINARY_NAME)
    deps:
            $(GOGET) github.com/markbates/goth
            $(GOGET) github.com/markbates/pop
    
    
    # Cross compilation
    build-linux:
            CGO_ENABLED=0 GOOS=linux GOARCH=amd64 $(GOBUILD) -o $(BINARY_UNIX) -v
    docker-build:
            docker run --rm -it -v "$(GOPATH)":/go -w /go/src/bitbucket.org/rsohlich/makepost golang:latest go build -o "$(BINARY_UNIX)" -v
```              

Let’s say I like to keep the idea of the DRY rule. So it is handy to declare commonly used commands and variables at the top of the file. And refer to them further in the file.

    # Basic go commands
    GOCMD=go
    GOBUILD=$(GOCMD) build
    GOCLEAN=$(GOCMD) clean
    GOTEST=$(GOCMD) test
    GOGET=$(GOCMD) get
    
    # Binary names
    BINARY_NAME=mybinary
    BINARY_UNIX=$(BINARY_NAME)_unix

The makefile tasks are defined bellow labels like “<name>:”. These are executed by make tool if the parameter for make is given. If no parameter is provided to make tool, the first task is executed. In this case the “all” task is executed, which consists of “test” and “build” tasks.


    > make run // call specific task
    > make // make tool calls "all" task

## Basic commands

One of the essential parts of the makefile is the build step. Using the variable $(GOBUILD) the “go build” command is executed. The binary name is defined by “-o $(BINARY_NAME)” and I found useful to switch to the verbose mode with “-v”. With verbose mode, you can see the currently built packages.


    build:
      $(GOBUILD) -o $(BINARY_NAME) -v // expands to: "go build -o mybinary -v"

Just because a lot of us are just lazy, the “run” task is here. The task builds the binary and executes the application consequently.


    run:
            $(GOBUILD) -o $(BINARY_NAME) -v ./...
            ./$(BINARY_NAME)

Naturally, the test command should be the part of the project makefile. I personally always choose the verbose switch to be able to debug and watch the test run better.


    test:
      $(GOTEST) -v ./...

If the project uses CI/CD or just for consistency, it is good to keep the list of dependencies used in packages. This is done by the “deps” task, which should get all the necessary dependencies by “go get” command.


    deps:
            $(GOGET) github.com/markbates/goth
            $(GOGET) github.com/markbates/pop

To wrap up this section of usual commands the clean command is accommodated into makefile. The “rm -f” command is added to remove binary with the custom name in $(BINARY_XXX) variable. Typically another clean-up command could be part of this make section.

    clean: 
            $(GOCLEAN)
            rm -f $(BINARY_NAME)
            rm -f $(BINARY_UNIX)


## Crosscompilation commands

If the project is intended to run on another platform than the one where it is developed, it is useful to include cross compilation commands to make file. I usually run the binaries on 
Linux platform in the container, so the makefile contains the Linux build.

    build-linux:
            CGO_ENABLED=0 GOOS=linux GOARCH=amd64 $(GOBUILD) -o $(BINARY_UNIX) -v

If your code use C binding you could get stuck a little bit. The issue with CGO is that you need a gcc compatible for given platform. So if the development is done on OSX/Windows you need to build gcc to be able to compile for Linux. At least for me, the configuration of gcc to cross compile the C code is not so straightforward on OSX. If the CGO is needed, the docker image is the best way how to create Linux build. The only requirement: Docker has to be installed.


    docker-build:
            docker run --rm -it -v "$(GOPATH)":/go -w /go/src/bitbucket.org/rsohlich/makepost golang:latest go build -o "$(BINARY_UNIX)" -v
## Summary

This way you can make your go development process more effective and fluent. The makefile used in this post is available here. Don’t hesitate to comment or ask questions, I’ll be happy to answer or talk about this. Enjoy!

