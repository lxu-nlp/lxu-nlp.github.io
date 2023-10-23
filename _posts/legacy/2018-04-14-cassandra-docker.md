---
layout: post
title: "Use Docker Container to Run Cassandra and Java Jar"
date: 2018-04-14
categories: [Programming]
tags: [java, cassandra, docker]
---

## What is Container?

I guess most software engineers are familiar with Docker Container already, and might be using it daily as part of CI/CD pipeline. Here is the concept of Container on the Docker official page:

> "A container image is a lightweight, stand-alone, executable package of a piece of software that includes everything needed to run it: code, runtime, system tools, system libraries, settings. Available for both Linux and Windows based apps, containerized software will always run the same, regardless of the environment."

Simply said, a container is a lightweight VM that packs up everything from OS to target application and is ready to run, separating environment from infrastructure.

Container, in conjunction with container orchestration tools such as Kubernetes (officially supported by Docker), or Apache Mesos + Marathon, has become the core part of cloud CI/CD deployment in most tech companies.

A **container** is an instance of an **image**.

## How to Run a Cassandra Image?

Let's say during development, we need to run Cassandra locally. Without Container, unless we want to spin up a whole VM just for Cassandra, the only option is to install the Cassandra on the local machine. However, with the power of Container, we can just simply pull a Cassandra image and run it directly.

Here is the official Docker Hub for Cassandra image: <https://hub.docker.com/_/cassandra/>

Let's run Cassandra by one-liner. (Assuming Docker is already installed on the local machine.)

```shell
docker run -p 127.0.0.1:32779:9042 --name cassandra -d cassandra:latest
```

`-p` means porting mapping. Cassandra runs on port 9042 for client connection. However, since Cassandra runs inside the container, we want to map the container port back to the port of our real local machine, so that we have a way to connect to the Cassandra instance. In this case, we map the container port 9042 to local machine port 32779, which enables us to connect to Cassandra by using port 32779. The mapped port can be any port as long as there are no conflicts.

`-d` means detach mode. We don't need interaction with Cassandra server directly, so we want it to run in the background. Later, we need CQL client container to run foreground (attach mode), because we need to interact with CQL client.

`cassandra:latest` is the unique identifier of the image we target to run. "cassandra" is the repository name, and "latest" is the tag which usually indicates the version.

Just by one line, the Cassandra is running locally!

The next step is to have a client connecting to the Cassandra. There are multiple GUI available for Cassandra client. Here I just use one to demo the connection. Notice that we use port 32779 for connection, as specified above.

![cassandra-connection](/assets/img/legacy/cassandra-connection.png)

Let's connect to Cassandra, create keyspace and table, and verify Cassandra is running.

![cassandra-connection](/assets/img/legacy/cassandra-table.png)

Looks good! We can also create another container with native CQL client to connect to this Cassandra container.

```shell
docker run -it --link cassandra:cassandra --rm cassandra cqlsh cassandra
```

Let's verify CQL client container is running by listing all keyspaces. The result should contain the keyspace "testspace" we just created above.

![cassandra-connection](/assets/img/legacy/cassandra-docker.png)

It is working!

## How to Create Docker Image That Can Run Jar?

Let's create our own Docker image. In the simplest case, we make the container to run a Java Jar file.

Building an image is like adding stuff layer by layer. Usually we build image based on a basic parent image, then we add things step by step. To continue, we need to have a basic understand of how to build an image: <https://docs.docker.com/develop/develop-images/baseimages/>

The most important thing is the Dockerfile. It is the instruction of how to build the docker image. Here we use the Java Jar from my previous post: <http://localhost:8888/2018/02/13/dropwizard-example/>

```dockerfile
FROM openjdk:8-jre-alpine3.7
WORKDIR /
ADD tryoutswagger-1.0-SNAPSHOT.jar tryoutswagger-1.0-SNAPSHOT.jar
ADD config.yml config.yml
ADD docker-entrypoint.sh docker-entrypoint.sh
RUN chmod a+x docker-entrypoint.sh
EXPOSE 8080
ENTRYPOINT ["/docker-entrypoint.sh"]
```

Here, *openjdk:8-jre-alpine3.7* is the base image we build upon. It is a popular lightweight image that contains JDK to run Java application.

`ADD` is the command to add our local files to the image. We should include all necessary files to add to the image. In our case, they are the Jar file, YAML file, and a script file to start our *tryoutswagger* application.

Since we are going to run the script file *docker-entrypoint.sh*, we want to change the permission to make it executable.

Next, since our *tryoutswagger* is a server application and listens on port 8080, we expose the port 8080 of the image to make it available.

The final step is the `ENTRYPOINT` command, which will be called when we start the container. The effect is once we start the container, the Java app will be run automatically, without any explicit command.

Here is the script file *docker-entrypoint.sh* that contains the command to run the Java application. Noted that we need to specify `/bin/sh`, because `bash` is not available in the base image *openjdk:8-jre-alpine3.7*.

```shell
#!/bin/sh
java -jar tryoutswagger-1.0-SNAPSHOT.jar server config.yml
```

Now that we have everything, let's build the image.

```shell
docker build -t tryoutswagger:1.0 .
```

Verify the image by listing all the images on our local machine.

Docker Build Image

Run this container.

```shell
docker run -p 58888:8080 --name javaapp -d tryoutswagger:1.0
```

The java app is running inside the container. Go to the swagger page by using the mapped port 58888.

![cassandra-connection](/assets/img/legacy/swagger-docker.png)

Looks good! The java app is running!
