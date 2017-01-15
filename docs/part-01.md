Part 1: Theory and a little practice
====================================

What is Docker?
---------------

Docker is an **open-source** project that **automates** the **deployment** of applications inside software **containers**. It has very developed eco-system to make, distribute and handle containers. Also it’s very well documented.

Docker uses the resource isolation features of the Linux kernel such as _cgroups_ and kernel _namespaces_, and a _union-capable file system_ such as OverlayFS and others to allow independent "containers" to run within a single Linux instance, avoiding the overhead of starting and maintaining virtual machines.

Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server. This guarantees that the software will always run the same, regardless of its environment.

Link: [What is Docker](https://www.docker.com/what-docker)

Why do we need it?
------------------

* Faster development process (using 3rd parties like PostgreSQL, Redis, Elasticsearch, whatever).
* Handy application encapsulation (you can deliver your application in one piece).
* Same behaviour on local machine / dev / stage / production servers.
* Easy and clear monitoring.
* Easy to scale (If you’ve done your application right it will be ready to scaling even not only in Docker).

Links: [Easy to develop](https://www.docker.com/what-docker#/Advantage) | [Easy to share](https://www.docker.com/what-docker#/easily_share) | [Easy to scale](https://www.docker.com/what-docker#/rapid_development)

What is the difference from virtualization?
-------------------------------------------

**Virtual machines** include the application, the necessary binaries and libraries, and an entire guest operating system - all of which can amount to tens of GBs.

**Docker containers** include the application and all of its dependencies - but share the kernel with other containers, running as isolated processes in user space on the host operating system. Docker containers are not tied to any specific infrastructure: they run on any computer, on any infrastructure, and in any cloud.

Link: [Comparing containers and virtual machines](https://www.docker.com/what-docker#/VM)

Supported platforms
-------------------

Because Docker is based on features of Linux kernel the only native platform is Linux. But there a lot native applications for other platforms such as MacOS and Windows. For this platforms Docker application is encapsulated into tiny virtual machine with Linux. The usability of this applications now is on very high level. Also there a lot of supplementary apps such as [Kitematic](https://kitematic.com/) or [Docker Machine](https://docs.docker.com/machine/overview/) which helps to install Docker on platforms different from Linux and operate with it.

Link: [Docker components](https://docs.docker.com/#/components)

Installation
------------

Link: [Installation](https://docs.docker.com/engine/getstarted/step_one/)

Terminology
-----------

**Container** - running instance with required application. Containers are always created from images. Container could expose ports and  volumes to interact with other containers or outer world. Container could be easily killed / removed and re-created again in a very short time.

Link: [Dockerizing](https://docs.docker.com/engine/tutorials/dockerizing/)

**Image** - basic element for every container. Uses copy-on-write model. During building images every step is cached and could be reused. Building images could take some time - it depends on particular image. While containers can be started from images very quickly.

Link: [Dockerfile - best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

**Port** - is a TCP/UDP port in it’s classical meaning. There are a lot of modes for networking in Docker containers, so to make things simpler let’s assume that ports could be exposed to outer world (accessible from host OS) or connected to other containers - accessible only from that containers and invisible from outer world.

**Volume** - the best explanation is shared folder. Volume keeps data outside of its container but accessible from this container or other connected containers. Data stored in volumes is persistent.

Link: [Managing data in Docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)

**Registry** - server that keeps images. Could be compared with git - you can pull image from registry to deploy it locally and you can push locally built images to registry (create new image or update image version on registry). Docker registry application is Open Source app like the main application - so you could deploy your private Docker Registry on any server you want.

Link: [Registry](https://docs.docker.com/registry/)

**Docker hub** - is a registry with web-interface. It keeps a lot of Docker images with different software. There are “official” Docker images (made by Docker team or in cooperation with original software vendors - don’t assume that it’s always official software vendors). It’s free to register and has payed options. You can have one private image per account and infinite amount of public images for free.

Link: [Docker Hub](https://hub.docker.com/explore/)

Example 1: hello world
----------------------

Let’s run our first container:

```
docker run ubuntu /bin/echo 'Hello world'
```

* **docker run** is command to run a container.
* **ubuntu** is the image you run, for example the Ubuntu operating system image. When you specify an image, Docker looks first for the image on your Docker host. If the image does not exist locally, then the image is pulled from the public image registry Docker Hub.
* **/bin/echo 'Hello world'** is the command to run inside the new container.

This container just prints `Hello world` and stops its execution. Not very big deal, yeah?. Let’s try to create an interactive shell inside Docker container:

```
docker run -i -t --rm ubuntu /bin/bash
```

* **-t** flag assigns a pseudo-tty or terminal inside the new container.
* **-i** flag allows you to make an interactive connection by grabbing the standard input (STDIN) of the container.
* **--rm** flag to automatically remove the container when it exits. By default containers are not deleted.

This container lives until we keep the shell session and dies when we exit from session (like SSH session to remote server). To keep container running we need to daemonize it:

```
docker run --name daemon -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

* **--name daemon** assign name `daemon` to new container. If you don’t specify a name explicitly Docker will generate it automatically and assign it to the container.
* **-d** flag runs the container in the background (daemonize it).

Let’s see what containers do we have now:

```
docker ps -a
```

* **docker ps** is command to command to list containers.
* **-a** show all containers (default shows just running).

Command shows us that we have two containers:
* Exited first container that printed 'Hello world' once with automatically generated name.
* Third container `daemon` which runs in background.

Also there is no second container with interactive shell because we’ve set `--rm` option and it was automatically deleted after execution.

Ok. Let’s see what daemon container is doing right now - let’s see the logs of it:

```
docker logs -f daemon
```

* **docker logs** is command to fetch the logs of a container.
* **-f** flag to follow the log output (works actually like `tail -f`).

Now let’s stop `daemon` container:

```
docker stop daemon
```

* **docker stop** is command to stop Docker container.

```
docker ps -a
```

Container it stopped. We can start it again:

```
docker start daemon
```

And ensure that it’s running:

```
docker ps -a
```

Now let’s stop it again and remove all the containers manually:

```
docker stop daemon
docker rm <your first container name>
docker rm daemon
```

Links: [docker run](https://docs.docker.com/engine/reference/commandline/run/) | [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) | [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) | [docker start](https://docs.docker.com/engine/reference/commandline/start/) | [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) | [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)

Example 2: nginx
----------------

Now let’s try to create and run more meaningful container like `nginx`.

```
cd examples/nginx
docker run -d --name test-nginx -v $(pwd):/usr/share/nginx/html:ro nginx:latest
```

Now you can check [this url](http://127.0.0.1/) url in your browser.

We can try to change `/example/nginx/index.html` (which is mounted as a volume to directory `/usr/share/nginx/html` inside container) and refresh the page.

Let’s investigate some information about `test-nginx` container:

```
docker inspect test-nginx
```

This command displays system wide information regarding the Docker installation. Information displayed includes the kernel version, number of containers and images, exposed ports, mounted volumes, etc.

Links: [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) | [Managing data in Docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)

Dockerfile
----------

To build a Docker image you need to write Dockerfile. It’s a plain text file with instructions and its arguments. Here is the description of instructions we’ll use in next example:

**FROM** - set base image
**RUN** - execute command in container
**ENV** - set environment variable
**WORKDIR** - set working directory
**VOLUME** - create mount-point for volume
**CMD** - set executable for container

Link: [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

Example 3: writing Dockerfile
-----------------------------

Let’s create an image that will get site contents with `curl` and store it to the text file. We’ll pass site url via environment variable `SITE_URL`. Resulting file will be placed in a directory mounted as volume.

```
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install --no-install-recommends --no-install-suggests -y curl
ENV SITE_URL http://nihera.net/
WORKDIR /data
VOLUME /data
CMD sh -c "curl -L $SITE_URL > /data/results"
```

Step into `examples/curl` directory and execute the following command to build an image:

```
docker build . -t test-curl
```

* **docker build** is a command to build a new image locally.
* **-t** set the name:tag to image.

Now we have our new image and we can see it in list of existing images:

```
docker images
```

So we can create and run container from image. Let’s try with default parameters:

```
docker run --rm -v $(pwd)/vol:/data/:rw test-curl
cat ./vol/results
```

Let’s try with `ya.ru`:

```
docker run --rm -e SITE_URL=ya.ru -v $(pwd)/vol:/data/:rw test-curl
cat ./vol/results
```

Link: [docker build](https://docs.docker.com/engine/reference/commandline/build/) | [docker images](https://docs.docker.com/engine/reference/commandline/images/)

Difference between images & containers
--------------------------------------

**Containers** are the running instances based on images.

**Images** are the sequence of instructions (layers) from Dockerfile and base image context.

| mode | Command | Layer |  |
|-------|---------------------|-------|---|
| **write** | **container goes here** | 2b064135b7f0 |  |
| read only | sh -c "curl -L nihera.net" | 701e38299831 | ⬆ |
| read only | ... | 5171cea75fa4 | ⬆ |
| read only | apt-get install --no-install-recommends --no-install-suggests -y curl | cc0f88ebb125 | ⬆ |
| read only | apt-get update | b9474e097082 | ⬆ |
| **read only** | **base image** | 447ff49a67a0 | ⬆ |

Best practices of creating images
---------------------------------

* Include only necessary context
* Use a `.dockerignore` file
* Avoid installing unnecessary packages
* Use cache
* Be careful with volumes
* Use environment variables (in `RUN`, `EXPOSE`, `VOLUME`)

Link: [Dockerfile - best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

Connection between containers
-----------------------------

**Docker compose** - is the only right way to connect containers with each other.

Link: [Docker compose - overview](https://docs.docker.com/compose/overview/)

Example 4: docker-compose Python + Redis
----------------------------------------

Step into directory `examples/compose` and execute the following command:

```
docker-compose --project-name app-test -f docker-compose.yml up
```

Open [this url](http://127.0.0.1:5000/) in your browser.

Docker way
----------

* 1 application = 1 container
* Run process in foreground (not in daemon mode)
* Keep data out of container
* No SSH
* No manual configurations (or actions) inside container
