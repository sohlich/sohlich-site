+++
title = "Deploy on Clever Cloud with Bitbucket Pipelines"
draft = false
tags = [
"Clever Cloud","Bitbucket","golang","docker","CI/CD"]
categories = [
    "cloud","golang"
]
date = "2017-08-31T05:08:10+02:00"
thumbnail="/img/blog/clever_cloud/back.jpg"

+++

There are a lot of CI tools that are available on the market. These are usualy free for open-source project. Recently I’ve come across the [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) feature that is free also for private ones, with limitation naturally. It is the CI/CD tool bounded to Bitbucket repository, that is based on containers.
In this post, I’d like to walk through the setup of Pipelines to deploy the build on [Clever Cloud](https://www.clever-cloud.com). The Clever Cloud is PaaS provider, which I have been using for about 2 years. They come up with a lot of platforms, but the best one and my favorite is the Docker. With this one, it is possible to deploy any Docker image.
For this post, I’ve chosen the Go application to be deployed on top of the Docker platform, but the same approach could be used with other languages.

## Repository content

First, we need to add a Dockerfile to application folder (for the simplicity). The most simple Dockerfile could look like following one. 

${PROJECT_FOLDER}/Dockerfile
```
    FROM golang:latest
    #copy the binary of app
    COPY linux_bin linux_bin
    EXPOSE 8080
    #execute the binary
    ENTRYPOINT ["./linux_bin"]
```

The important step in Dockerfile is the exposing of port 8080. The Clever Cloud requires this port as a default HTTP port. If the application does not respond on this port, the monitor tool evaluates the application as non-working and the deployment fails.

The binary itself will be built during the Pipelines run. We can test the Dockerfile by building the binary. As we are running the binary within the container, it must be compiled for Linux and amd64 architecture.

```
    > GOOS=linux GOARCH=AMD64 go build -o linux_bin -v
    > docker build --rm -t clever_cloud_deploy .
    Sending build context to Docker daemon  42.55MB
    Step 1/4 : FROM golang:alpine
    Step 2/4 : COPY linux_bin linux_bin
    Step 3/4 : EXPOSE 8080
    Step 4/4 : CMD ./linux_bin
    Successfully built 692db10e1840
    Successfully tagged clever_cloud_deploy:latest
    > docker run --rm -it clever_cloud_deploy:latest
```

So the Dockerfile is OK. Now the Pipelines config file must be created. The easiest way how to do that is to copy the config from bitbucket docs and modify it to fit your needs. ([Bitbucket Docs](https://confluence.atlassian.com/bitbucket/language-guides-856821477.html)) 

${PROJECT_FOLDER}/bitbucket-pipelines.yml
```
    image: golang:latest
    pipelines:
      default:
        - step:
            caches:
              - gocache        # custom golang cache
            script: # Modify the commands below to build your repository.
              - PACKAGE_PATH="${GOPATH}/src/bitbucket.org/${BITBUCKET_REPO_OWNER}/${BITBUCKET_REPO_SLUG}"
              - mkdir -pv "${PACKAGE_PATH}"
              - tar -cO --exclude-vcs --exclude=bitbucket-pipelines.yml . | tar -xv -C "${PACKAGE_PATH}"
              - cd "${PACKAGE_PATH}"
              - go get -v
              - go build -o linux_bin -v
              - go test -v

```

## Deployment to Clever Cloud

The deployment of an application to Clever Cloud is done via push to the git repository. The push to the master branch invokes the deployment on the provider side. The Docker platform was chosen so the Dockerfile must be the essential part of pushed changes.  Naturally, all the files used in Dockerfile are needed. The deployment on the provider side, in fact, consists of the Docker file build and start of the container based on the created docker image.

### SSH Keys

To be able to push into the Clever Cloud repository, the SSH public key must be provided. The existing key could be added or you can generate a new one in Bitbucket repository (Settings → Pipelines → SSHKeys) and configure it to be allowed to use in Clever Cloud (see Clever Cloud settings).

### Clever Cloud repository and config

Each application in Clever Cloud has so called “deployment URL” defined in information tab in the Clever Cloud console (administration page). The URL is similar to one below.

```
    git+ssh://git@push-par-clevercloud-customers.services.clever-cloud.com/app_b5a2def7-ed24-4fde-bcf2-5d0525c2d00b.git
```
This URL would be used in Pipelines config file as a remote git repository to push the built files. The deploy than will consist of these steps:

- initialize Git repository
- commit all files needed for Dockerfile build
- push to remote master branch

Before we do that we need to add the push-par-clevercloud-customers.services.clever-cloud.com host to “known hosts”. This is done via Settings → SSH → Known Hosts section. 
After this is done the steps described above can be added to Pipelines config file.

${PROJECT_FOLDER}/bitbucket-pipelines.yml (partial)
```
              - git init
              - git config user.name "Pipeline" && git config user.email "pipeline@example.org"
              # Use -f switch to override .gitignore file
              - git add -f linux_bin 
              - git commit -m "init"
```

Initialisation of the repository, configuration of user and email is needed. Without this git refuses to work properly.

Finally, push all the changes to Clever Cloud repo using the —force switch.

```
    git push git+ssh://git@push-par-clevercloud-customers.services.clever-cloud.com/app_b5a2def7-ed24-4fde-bcf2-5d0525c2d00b.git master --force
```

The complete Pipelines configuration file could look like this.

${PROJECT_FOLDER}/bitbucket-pipelines.yml
```
    image: golang:latest
    pipelines:
      default:
        - step:
            caches:
              - gocache        # custom golang cache
              - xchache        # custom golang cache
              - pkgincache        # custom golang cache
            script: # Modify the commands below to build your repository.
              - PACKAGE_PATH="${GOPATH}/src/bitbucket.org/${BITBUCKET_REPO_OWNER}/${BITBUCKET_REPO_SLUG}"
              - mkdir -pv "${PACKAGE_PATH}"
              - tar -cO --exclude-vcs --exclude=bitbucket-pipelines.yml . | tar -xv -C "${PACKAGE_PATH}"
              - cd "${PACKAGE_PATH}"
              - go get -v
              - go build -o linux_bin -v
              - go test -v
              - git init
              # Deployment to clever cloud
              - git config user.name "Pipeline" && git config user.email "pipeline@example.org"
              - git add -f linux_bin
              - git commit -m "init"
              - git push git+ssh://git@push-par-clevercloud-customers.services.clever-cloud.com/app_b5a2def7-ed24-4fde-bcf2-5d0525c2d00b.git master --force
```

That’s it. After your commit arrive to Bitbucket repository the Pipelines run is started, subsequently the build deployed on Clever Cloud. Enjoy!

Link to the sample repository:
https://bitbucket.org/rsohlich/clever_cloud_deploy
