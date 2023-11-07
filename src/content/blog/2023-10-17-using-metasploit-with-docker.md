---
title: "Using Metasploit with Docker" 
pubDate: "Oct 17 2023"
description: "How to use metasploit with docker."
tags: ["security", "docker", "metasploit"]
heroImage: "/blog/2023-10-17-using-metasploit-with-docker/logo.png"
---

Metasploit framework is an open source software for the purpose of analysing and exploiting vulnerabilities. Over time it has become one of the most widely used tools.

In this case we will use this software, encapsulating everything in a docker container. This way, we have certain advantages. As it will not be necessary to install anything on our host system. We will also be able to use this software in projects that involve networks and docker containers. Below we can see the image we will use from Dockerhub.

![Imagen](/blog/2023-10-17-using-metasploit-with-docker/dockerhub.png)

To download the image we execute the command suggested by "Dockerhub".

```sh
$ docker pull metasploitframework/metasploit-framework
```

Now we create a container using the downloaded image. Note that it is only necessary to expose port "55553" if we plan to control Metasploit using the RPC API from outside docker. Also the parameters "-it" and "--rm" are not necessary if respectively we don't want to have an open terminal of the container and if we don't want the container to be deleted on stop.

```sh
$ docker run -it --rm -p 55553:55553 --name metasploit2 metasploitframework/metasploit-framework
```

If all goes well, we should have a terminal like this one:

![Imagen](/blog/2023-10-17-using-metasploit-with-docker/sample-console.png)

We are now ready to use metasploit from our container.

Another interesting possible scenario could be to attack elements that are on a network of our host system. In that case, it would be as simple as running metasploit with a network of type "HOST". This way we will have access to the network interfaces of our host system.

```sh
$ docker run -it --rm --network host --name metasploit2 metasploitframework/metasploit-framework
```

By simply running the "ifconfig" command from the metasploit terminal, we will see that we have the same interfaces as our host system.

Finally, we will also see how to automate the deployment of this container using a "Docker compose" file. Again, it is only necessary to expose port 55553 if we want to use the RFC API from outside. Note that this container will only be able to exploit systems with which it shares a network.

```yml
version: '3'
services:
  metasploit:
    image: metasploitframework/metasploit-framework:latest
    container_name: metasploit
    command: tail -f /dev/null
    ports:
      - "55553:55553"
    networks:
      - msfnet

networks:
  msfnet:

```

We run the following command, from the same directory as the docker-compose.yml file.

```sh
$ docker-compose up -d
```

And with this, now we have our metasploit container ready!
