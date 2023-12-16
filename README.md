# ed-docker-best-practice
by article https://medium.com/@fsegredo2000/docker-best-practices-6fa3de5f17cb

TL: DR

- Tips for Run Command, apt-get
- Image flavors
- COPY vs ADD
- Bonus (Dive)

# “How can I improve my DockerFile?”

Today, we’re going to look at a few things that can help you improve your Dockerfile.

## Reduce image size

This is a crucial aspect, as we do not wish to end up with excessively large images. By paying attention to how we install the packages that we need in the container, we can significantly reduce the size of the images.

- When using apt-get install certain package managers by default will install recommended packages, dependencies and debug packages (associated with the packages you pretend to install), which might not re required/usefull for your image, in that case we can cut down some size just by adding the flag — no-install-recommends

We can check the dependencies and recommended packages with the command:

```
apt-cache depends (PACKAGE)
```

![image](https://github.com/GrytsenkoAndrey/ed-docker-best-practice/assets/63291871/ed9e5bdf-4e63-4b54-9764-a6819a8f3ab1)


```
#Befor
RUN apt-get -y update &&  \
    apt-get install -y python
```
```
#After
RUN apt-get -y update &&  \ 
    apt-get install --no-install-recommends  \
    -y python
```

- Remove cached data that might endup in your image

```
RUN apt-get -y update &&  \ 
    apt-get install --no-install-recommends  \
    -y python && \
    rm -rf /var/lib/apt/lists/*
```


## RUN Command

- It is recommended to utilize a single RUN command to install multiple packages, as this method prevents the creation of additional layers of images that are not necessary.
- You can make your file more readable and maintainable by splitting long or complex instructions with backlash (\).

(for simplicity we will remove the — no-install-recomends and cleaning of cached data)

```
RUN apt-get -y updater && apt-get install -y \
  ca-certificates \
  coreutils \
  curl \
  openjdk11 \
  tzdata
```

- Always use the apt-get update and install together on the same instruction to avoid cashing issues, this is called Cache Busting

If we use separate instructions as Below, if the build is sucessfull all the layers (except the layer FROM) will be cached

```
FROM ubuntu:23.10
RUN apt-get update (CACHED)
RUN apt-get install -y curl (CACHED)
```

Eventually we need an extra package “bridge-utils” so we update the dockerfile and Build the image

```
FROM ubuntu:23.10
RUN apt-get update (CACHED)
RUN apt-get install -y  \ 
  curl \
  bridge-utils
```

Since the first layer is cached, docker will use the cached layer, which means the command “apt-get update” will be skipped and eventually you might end up working with outdated packages. Combining the instructions together forces the apt-get update to always run to make sure you’re working with the latest ones.

```
FROM ubuntu:23.10
RUN apt-get update && apt-get install -y  \ 
  curl=1.2.3.4 \
  bridge-utils
```

> what if I need a specific version for a package let’s say, curl always need to be on version 1.2.3.4?

That’s called version pinning and we can achive it doing the following:

```
FROM ubuntu:23.10
RUN apt-get update 
RUN apt-get install -y  \ 
  curl=1.2.3.4 \
  bridge-utils
```

## Minimal Images (Flavors)

Consider using the smallest possible versions of the image whenever possible. A flavor image is stripped of certain functions in order to make a smaller version of a base image. There are many functions that original/base images provide that you won’t need.

Consider de size diferent of a base and flavor image.

![image](https://github.com/GrytsenkoAndrey/ed-docker-best-practice/assets/63291871/08e7789b-88e9-408b-9600-2041abd52e9e)

## Use of Labels

Labels can be used to identify the author, maintainers of the image, or to share any information you see fit when publishing or sharing the image in public or private registries.

Keep in mind that the instruction LABEL won’t add extra layers to your image.

```
LABEL release-date="2023-02-12" owner="Secret"
```


## COPY VS ADD

Both instructions serve the same purpose, which is to copy files and directories to a docker image from the host.

```
COPY <src> <dest>

ADD <src> <dest>

Examples:
ADD /home/user/app /opt/app
ADD /home/user/app/docs.tar.gz /opt/app/docs
ADD https://sourcefile/file.pdf /opt/app/
```

The main difference is that ADD has extra features:

- Extracting local-only compressed files (only supported compressed formats — Identity, gzip, bzip2 or xz)
- Download files from Remote URL’s

However, as our primary concern is image size and avoiding the addition of layers, ADD might not be the best use case.

Let’s see the Do’s and Dont’s

- Docker strongly discourages the use of ADDto fetch packages from remote URL’s and recommends curl or wget to accomplish the same result with better efficiency.

In this particular example, we acquire our file, extract it to the destination folder, and execute the make command. Each instruction adds layers to your image, resulting in a total of three additional layers, resulting in a larger image.

```
ADD https://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

The docker recommended way

```
RUN mkdir -p /usr/src/things \
    && curl -SL https://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

Let’s recap: COPY should be your go-to for copying files and directories as it is more transparent, and try to use Linux commands to download and extract files instead of ADD

```
DON'T -> ADD  /home/user/application /opt/app
DO    -> COPY /home/user/application /opt/app
```

Unique use cases for ADD

- Local tar file auto-extraction into an image

```
ADD rootfs.tar.xz /
```

Keeping these points in mind will make your image much more efficient.

1 — With the COPYand ADD instructions you can modify the owner and mode of the files and directories while copying them to the image.

```
COPY --chown=55:mygroup files* /somedir/
COPY --chown=myuser:mygroup --chmod=644 files* /somedir/

ADD --chown=55:mygroup files* /somedir/
ADD --chown=myuser:mygroup --chmod=655 files* /somedir/
```

Tags

Use a specific tag when using an image. Using the latest tag might give you some unexpected problems and might break changes over time. By utilizing this method, it is possible to ensure compatibility and the utilization of accurate versions of a particular image.

```
FROM docker_image:1.0.0
```

instead of

```
FROM docker.image:latest
```





