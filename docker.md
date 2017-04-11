
* Lab 1: Our First Containers
* Lab 2: Background Containers
* Lab 3: Restarting and Attaching to Containers
* Lab 4: Understanding Docker Images
* Lab 5: Building Images Interactively
* Lab 6: Building Docker images
* Lab 7: CMD and ENTRYPOINT
* Lab 8: Copying files during the build
* Lab 9: Advanced Dockerfiles

## Lab 1: Our First Containers

#### Objectives

At the end of this lab, you will have:

-   Seen Docker in action.

-   Started your first containers.


#### Docker architecture

Docker is a client-server application.

-   **The Docker Engine (or "daemon")**

 Receives and processes incoming Docker API requests.

-   **The Docker client**

 Talks to the Docker daemon via the Docker API.

 We'll use mostly the CLI embedded within the docker binary.

-   **Docker Hub Registry**

 Collection of public images.

 The Docker daemon talks to it via the registry API.

#### Hello World

In your Docker environment, just run the following command:
```
$ docker run busybox echo hello world
hello world
```
That was our first container!

-   We used one of the smallest, simplest images available: busybox.

-   busybox is typically used in embedded systems (phones, routers...)

-   We ran a single process and echo'ed hello world.

A more useful container

Let's run a more exciting container:
```
$ docker run -it ubuntu
```
-   This is a brand new container.

-   It runs a bare-bones, no-frills ubuntu system.

 • `-it` is shorthand for `-i -t`.

-   `-i` tells Docker to connect us to the container's stdin.

-   `-t` tells Docker that we want a pseudo-terminal.


Do something in our container

Try to run figlet in our container.
```
root@04c0bb0a6c07:/\# figlet hello
bash: figlet: command not found
```
Alright, we need to install it.


An obvservation

Let's check how many packages are installed here.
```
root@04c0bb0a6c07:/
\# dpkg -l | wc -l 189
```
 • `dpkg -l `lists the packages installed in our container

-   `wc -l` counts them

-   If you have a Debian or Ubuntu machine, you can run the same command and compare the results.


We want figlet, so let's install it:
```
root@04c0bb0a6c07:/\# apt-get update

...

Fetched 1514 kB in 14s (103 kB/s) Reading package lists... Done

root@04c0bb0a6c07:/\# apt-get install figlet Reading package lists... Done

...
```
One minute later, figlet is installed!
```
 # figlet hello _ __
    | |__   ___| | | ___
    | '_ \ / _ \ | |/ _ \
    | | | |  __/ | | (_) |
    |_| |_|\___|_|_|\___/
```

Exiting our container

Just exit the shell, like you would usually do.

(E.g. with ^D or exit)

 root@04c0bb0a6c07:/\# exit

-   Our container is now in a *stopped* state.

-   It still exists on disk, but all compute resources have been freed up.


What if we start a new container, and try to run figlet again?
```
$ docker run -it ubuntu root@b13c164401fb:/\# figlet bash: figlet: command not found
```
-   We started a *brand new container*.

-   The basic Ubuntu image was used, and figlet is not here.

-   We will see in the next chapters how to bake a custom image with figlet.


## Lab 2: Background Containers

#### Objectives

Our first containers were *interactive*.

We will now see how to:

-   Run a non-interactive container.

-   Run a container in the background.

-   List running containers.

-   Check the logs of a container.

-   Stop a container.

-   List stopped containers.


 A non-interactive container

We will run a small custom container.

This container just displays the time every second.
```
 $ docker run jpetazzo/clock Fri Feb 20 00:28:53 UTC Fri Feb 20 00:28:54 UTC Fri Feb 20 00:28:55 UTC
 ...
```
-   This container will run forever.

-   To stop it, press ^C.

-   Docker has automatically downloaded the image jpetazzo/clock.

-   This image is a user image, created by jpetazzo.

-   We will hear more about user images (and other types of images) later.


 Run a container in the background

Containers can be started in the background, with the -d flag (daemon mode):
```
 $ docker run -d jpetazzo/clock 47d677dcfba4277c6cc68fcaa51f932b544cab1a187c853b7d0caf4e8debe5ad
```
-   We don't see the output of the container.

-   But don't worry: Docker collects that output and logs it!

-   Docker gives us the ID of the container.


 List running containers

How can we check that our container is still running?

With docker ps, just like the UNIX ps command, lists running processes.
```
| $ docker ps  | IMAGE            | ... | CREATED         | STATUS           | ... |     |
|--------------|------------------|-----|-----------------|------------------|-----|-----|
| CONTAINER ID |                  |     |                 |                  |     |     |
| 47d677dcfba4 | jpetazzo/clock   | ... | 2 minutes ago   | Up 2 minutes ... |     |
```
Docker tells us:

-   The (truncated) ID of our container.

-   The image used to start the container.

-   That our container has been running (Up) for a couple of minutes.

-   Other information (COMMAND, PORTS, NAMES) that we will explain later.


 Starting more containers

Let's start two more containers.
```
 $ docker run -d jpetazzo/clock 57ad9bdfc06bb4407c47220cf59ce21585dce9a1298d7a67488359aeaea8ae2a $ docker run -d jpetazzo/clock 068cc994ffd0190bbe025ba74e4c0771a5d8f14734af772ddee8dc1aaf20567d
```
Check that docker ps correctly reports all 3 containers.


Two useful flags for docker ps

To see only the last container that was started:
```
| $ docker ps -l | IMAGE          | ... | CREATED         | STATUS           | ... |     |
|----------------|----------------|-----|-----------------|------------------|-----|-----|
| CONTAINER ID   |                |     |                 |                  |     |     |
| 068cc994ffd0   | jpetazzo/clock | ... | 2 minutes ago   | Up 2 minutes ... |     |
```
To see only the ID of containers:
```
 $ docker ps -q 068cc994ffd0 57ad9bdfc06b 47d677dcfba4
```
Combine those flags to see only the ID of the last container started!
```
 $ docker ps -lq 068cc994ffd0
```

 View the logs of a container

We told you that Docker was logging the container output.

Let's see that now.
```
 $ docker logs 068
 Fri Feb 20 00:39:52 UTC Fri Feb 20 00:39:53 UTC
 ...
```
-   We specified a *prefix* of the full container ID.

-   You can, of course, specify the full ID.

-   The logs command will output the *entire* logs of the container. (Sometimes, that will be too much. Let's see how to address that.)

#### View only the tail of the logs

To avoid being spammed with eleventy pages of output, we can use the --tail option:
```
 $ docker logs --tail 3 068 Fri Feb 20 00:55:35 UTC Fri Feb 20 00:55:36 UTC Fri Feb 20 00:55:37 UTC
```
-   The parameter is the number of lines that we want to see.


 Follow the logs in real time

Just like with the standard UNIX command tail -f, we can follow the logs of our container:
```
 $ docker logs --tail 1 --follow 068 Fri Feb 20 00:57:12 UTC
 Fri Feb 20 00:57:13 UTC ^C
```
-   This will display the last line in the log file.

-   Then, it will continue to display the logs in real time.

-   Use ^C to exit.


 Stop our container

There are two ways we can terminate our detached container.

 • Killing it using the docker kill command.

-   Stopping it using the docker stop command.

The first one stops the container immediately, by using the KILL signal.

The second one is more graceful. It sends a TERM signal, and after 10 seconds, if the container has not stopped, it sends KILL.

Reminder: the KILL signal cannot be intercepted, and will forcibly terminate the container.


 Stopping our containers

Let's stop one of those containers:
```
 $ docker stop 47d6 47d6
```
This will take 10 seconds:

-   Docker sends the TERM signal;

-   the container doesn't react to this signal (it's a simple Shell script with no special signal handling);

-   10 seconds later, since the container is still running, Docker sends the KILL signal;

-   this terminates the container.


 Killing the remaining containers

Let's be less patient with the two other containers:
```
 $ docker kill 068 57ad 068
 57ad
```
The stop and kill commands can take multiple container IDs.

Those containers will be terminated immediately (without the 10 seconds delay). Let's check that our containers don't show up anymore:
```
 $ docker ps
```

 List stopped containers

We can also see stopped containers, with the -a (--all) option.
```
| $ docker ps -a | IMAGE          | ... | CREATED   | STATUS     |                          |               |            |
|----------------|----------------|-----|-----------|------------|--------------------------|---------------|------------|
| CONTAINER ID   |                |     |           |            |                          | min. ago      |            |
| 068cc994ffd0   | jpetazzo/clock | ... | 21        | min. ago   | Exited (137) 3           |               |            |
| 57ad9bdfc06b   | jpetazzo/clock | ... | 21        | min. ago   | Exited (137)             | 3             | min. ago   |
| 47d677dcfba4   | jpetazzo/clock | ... | 23        | min. ago   | Exited (137)             | 3             | min. ago   |
| 5c1dfd4d81f1   | jpetazzo/clock | ... | 40        | min. ago   | Exited (0) 40 min. ago   |               |
| b13c164401fb   | ubuntu         | ... | 55        | min. ago   | Exited (130)             | 53 min. ago   |            |
```
## Lab 3: Restarting and Attaching to Containers

#### Objectives

We have started containers in the foreground, and in the background. In this chapter, we will see how to:

-   Put a container in the background.

-   Attach to a background container to bring it to the foreground.

-   Restart a stopped container.


 Background and foreground

The distinction between foreground and background containers is arbitrary. From Docker's point of view, all containers are the same.

All containers run the same way, whether there is a client attached to them or not. It is always possible to detach from a container, and to reattach to a container.

Analogy: attaching to a container is like plugging a keyboard and screen to a physical server.


 Detaching from a container

-   If you have started an *interactive* container (with option -it), you can detach from it.

-   The "detach" sequence is ^P^Q.

-   Otherwise you can detach by killing the Docker client.

 (But not by hitting ^C, as this would deliver SIGINT to the container.)

What does -it stand for?

-   -t means "allocate a terminal."

-   -i means "connect stdin to the terminal."

Specifying a custom detach sequence

-   You don't like ^P^Q? No problem!

 • You can change the sequence with docker run --detach-keys.

 • This can also be passed as a global option to the engine.

Start a container with a custom detach command:
```
 $ docker run -ti --detach-keys ctrl-x,x jpetazzo/clock
```
Detach by hitting ^X x. (This is ctrl-x then x, not ctrl-x twice!)

Check that our container is still running:
```
 $ docker ps -l
```

 Attaching to a container

You can attach to a container:
```
 $ docker attach <containerID>
```
-   The container must be running.

-   There *can* be multiple clients attached to the same container.

-   If you don't specify --detach-keys when attaching, it defaults back to ^P^Q.

Try it on our previous container:
```
 $ docker attach $(docker ps -lq)
```
Check that ^X x doesn't work, but ^P ^Q does.

 Detaching from non-interactive containers

-   **Warning:** if the container was started without -it...

    -   You won't be able to detach with ^P^Q.

    -   If you hit ^C, the signal will be proxied to the container.

-   Remember: you can always detach by killing the Docker client.


 Checking container output

• Use docker attach if you intend to send input to the container.

• If you just want to see the output of a container, use docker logs.
```
$ docker logs --tail 1 --follow <containerID>
```
 Restarting a container

When a container has exited, it is in stopped state.

It can then be restarted with the start command.
```
 $ docker start <yourContainerID>
```
The container will be restarted using the same options you launched it with.

You can re-attach to it if you want to interact with it:
```
 $ docker attach <yourContainerID>
```
Use `docker ps -a` to identify the container ID of a previous jpetazzo/clock container, and try those commands.

 Attaching to a REPL

-   REPL = Read Eval Print Loop

-   Shells, interpreters, TUI ...

-   Symptom: you docker attach, and see nothing

-   The REPL doesn't know that you just attached, and doesn't print anything

-   Try hitting ^L or Enter

 SIGWINCH

-   When you docker attach, the Docker Engine sends a couple of SIGWINCH signals to the container.

-   SIGWINCH = WINdow CHange; indicates a change in window size.

-   This will cause some CLI and TUI programs to redraw the screen.

-   But not all of them.


## Lab 4: Understanding Docker Images

#### Objectives

In this lab, we will explain:

-   What is an image.

-   What is a layer.

-   The various image namespaces.

-   How to search and download images.

-   Image tags and when to use them.

 What is an image?

-   An image is a collection of files + some meta data.

 (Technically: those files form the root filesystem of a container.)

-   Images are made of *layers*, conceptually stacked on top of each other.

-   Each layer can add, change, and remove files.

-   Images can share layers to optimize disk usage, transfer times, and memory use.

-   Example:

    -   CentOS

    -   JRE

    -   Tomcat

    -   Dependencies

    -   Application JAR

    -   Configuration

 Differences between containers and images

-   An image is a read-only filesystem.

-   A container is an encapsulated set of processes running in a read-write copy of that filesystem.

-   To optimize container boot time, *copy-on-write* is used instead of regular copy.

-   docker run starts a container from a given image.

Let's give a couple of metaphors to illustrate those concepts.

 Image as stencils

Images are like templates or stencils that you can create containers from.


Object-oriented programming

-   Images are conceptually similar to *classes*.

-   Layers are conceptually similar to *inheritance*.

-   Containers are conceptually similar to *instances*.

 Wait a minute...

If an image is read-only, how do we change it?

-   We don't.

-   We create a new container from that image.

-   Then we make changes to that container.

-   When we are satisfied with those changes, we transform them into a new layer.

-   A new image is created by stacking the new layer on top of the old image.

 A chicken-and-egg problem

-   The only way to create an image is by "freezing" a container.

-   The only way to create a container is by instanciating an image.

-   Help!

Creating the first images

There is a special empty image called scratch.

-   It allows to *build from scratch*.

The docker import command loads a tarball into Docker.

-   The imported tarball becomes a standalone image.

-   That new image has a single layer.

Note: you will probably never have to do this yourself.

 Creating other images

docker commit

-   Saves all the changes made to a container into a new layer.

-   Creates a new image (effectively a copy of the container).

docker build

-   Performs a repeatable build sequence.

-   This is the preferred method!

We will explain both methods in a moment.

 Images namespaces

There are three namespaces:

-   Official images

 e.g. ubuntu, busybox ...

-   User (and organizations) images e.g. jpetazzo/clock

-   Self-hosted images

 e.g. registry.example.com:5000/my-private/image

Let's explain each of them.

 Root namespace

The root namespace is for official images. They are put there by ., but they are generally authored and maintained by third parties.

Those images include:

-   Small, "swiss-army-knife" images like busybox.

-   Distro images to be used as bases for your builds, like ubuntu, fedora...

-   Ready-to-use components and services, like redis, postgresql... User namespace

The user namespace holds images for Docker Hub users and organizations. For example:

 jpetazzo/clock

The Docker Hub user is:

 jpetazzo

The image name is:

 clock

 Self-Hosted namespace

This namespace holds images which are not hosted on Docker Hub, but on third party registries.

They contain the hostname (or IP address), and optionally the port, of the registry server.

For example:

 localhost:5000/wordpress

-   localhost:5000 is the host and port of the registry

-   wordpress is the name of the image

 How do you store and manage images?

Images can be stored:

-   On your Docker host.

-   In a Docker registry.

You can use the Docker client to download (pull) or upload (push) images.

To be more accurate: you can use the Docker client to tell a Docker server to push and pull images to and from a registry.

 Showing current images

Let's look at what images are on our host now.
```
| $ docker images | TAG      | IMAGE ID       | CREATED         | SIZE         |          |     |
|-----------------|----------|----------------|-----------------|--------------|----------|-----|
| REPOSITORY      |          |                |                 |              | MB       |     |
| fedora          | latest   | ddd5c9c1d0f2   | 3               | days ago     | 204.7    |     |
| centos          | latest   | d0e7f81ca65c   | 3               | days ago     | 196.6    | MB  |
| ubuntu          | latest   | 07c86167cdc4   | 4               | days ago     | 188 MB   | MB  |
| redis           | latest   | 4f5f397d4b7c   | 5               | days ago     | 177.6    |     |
| postgres        | latest   | afe2b5e1859b   | 5               | days ago     | 264.5    | MB  |
| alpine          | latest   | 70c557e50ed6   | 5               | days ago     | 4.798    | MB  |
| debian          | latest   | f50f9524513f   | 6               | days ago     | 125.1    | MB  |
| busybox         | latest   | 3240943c9ea3   | 2               | weeks ago    | 1.114    | MB  |
| training/namer  | latest   | 902673acc741   | 9               | months ago   | 289.3    | MB  |
| jpetazzo/clock  | latest   | 12068b93616f   | 12 months ago   | 2.433        | MB       |     |
```
 Searching for images

We cannot list *all* images on a remote registry, but we can search for a specific keyword:
```
| $ docker search zookeeper | STARS OFFICIAL              | AUTOMATED   |          |
|---------------------------|-----------------------------|-------------|----------|-----|
| NAME                      | DESCRIPTION                 |             |          |     |
| jplock/zookeeper          | Builds a docker image ...   | 103         | \[OK\]   |     |
| mesoscloud/zookeeper      | ZooKeeper                   | 42          | \[OK\]   |     |
| springxd/zookeeper        | A Docker image that ca...   | 5           | \[OK\]   |     |
| elevy/zookeeper           | ZooKeeper configured t...   | 3           | \[OK\]   |     |
```
-   "Stars" indicate the popularity of the image.

-   "Official" images are those in the root namespace.

-   "Automated" images are built automatically by the Docker Hub.

 (This means that their build recipe is always available.)

 Downloading images

There are two ways to download images.

 • Explicitly, with docker pull.

-   Implicitly, when executing docker run and the image is not found locally.

 Pulling an image
```
$ docker pull debian:jessie Pulling repository debian b164861940b8: Download complete
b164861940b8: Pulling image (jessie) from debian d1881793a057: Download complete
```
-   As seen previously, images are made up of layers.

-   Docker has downloaded all the necessary layers.

-   In this example, :jessie indicates which exact version of Debian we would like. It is a *version tag*.

 Image and tags

-   Images can have tags.

-   Tags define image versions or variants.

 • docker pull ubuntu will refer to ubuntu:latest.

-   The :latest tag is generally updated often.

 When to (not) use tags

Don't specify tags:

-   When doing rapid testing and prototyping.

-   When experimenting.

-   When you want the latest version.

Do specify tags:

-   When recording a procedure into a script.

-   When going to production.

-   To ensure that the same version will be used everywhere.

-   To ensure repeatability later.

Section summary

We've learned how to:

-   Understand images and layers.

-   Understand Docker image namespacing.

-   Search and download images.

## Lab 5: Building Images Interactively

#### Objectives

In this lab, we will create our first container image.

It will be a basic distribution image, but we will pre-install the package figlet.

We will:

-   Create a container from a base image.

-   Install software manually in the container, and turn it into a new image.

-   Learn about new commands: docker commit, docker tag, and docker diff.

 Building Images Interactively

As we have seen, the images on the Docker Hub are sometimes very basic. How do we want to construct our own images?

As an example, we will build an image that has figlet.

First, we will do it manually with docker commit.

Then, in an upcoming chapter, we will use a Dockerfile and docker build.

Building from a base

Our base will be the ubuntu image.

 Create a new container and make some changes

Start an Ubuntu container:
```
 $ docker run -it ubuntu
 root@<yourContainerId>:\#/
```
Run the command apt-get update to refresh the list of packages available to install.

Then run the command apt-get install figlet to install the program we are interested in.
```
 root@<yourContainerId>:\#/ apt-get update && apt-get install figlet

 .... OUTPUT OF APT-GET COMMANDS ....
```
 Inspect the changes

Type exit at the container prompt to leave the interactive session.

Now let's run docker diff to see the difference between the base image and our container.
```
 $ docker diff <yourContainerId>
 C /root
 A /root/.bash\_history C /tmp
 C /usr
 C /usr/bin
 A /usr/bin/figlet
 ...
```
 Docker tracks filesystem changes

As explained before:

-   An image is read-only.

-   When we make changes, they happen in a copy of the image.

-   Docker can show the difference between the image, and its copy.

-   For performance, Docker uses copy-on-write systems.

 (i.e. starting a container based on a big image doesn't incur a huge copy.)

 Commit and run your image

The docker commit command will create a new layer with those changes, and a new image using this new layer.
```
 $ docker commit <yourContainerId> <newImageId>
```
The output of the docker commit command will be the ID for your newly created image.

We can run this image:
```
 $ docker run -it <newImageId>
 root@fcfb62f0bfde:/\# figlet hello

| |__   ___| | | ___
| '_ \ / _ \ | |/_ \
| | | | __/ | | (_) |
|_| |_|\___|_|_|\___/
```

**Tagging images**

Referring to an image by its ID is not convenient. Let's tag it instead. We can use the tag command:
```
 $ docker tag <newImageId> figlet
```
But we can also specify the tag as an extra argument to commit:
```
 $ docker commit <containerId> figlet
```
And then run it using its tag:
```
 $ docker run -it figlet
```
 What's next?

Manual process = bad.

Automated process = good.

In the next lab, we will learn how to automate the build process by writing a Dockerfile.

## Lab 6: Building Images With A Dockerfile

#### Objectives

We will build a container image automatically, with a Dockerfile.

At the end of this lab, you will be able to:

-   Write a Dockerfile.

-   Build an image from a Dockerfile.

 Dockerfile overview

-   A Dockerfile is a build recipe for a Docker image.

-   It contains a series of instructions telling Docker how an image is constructed.

-   The docker build command builds an image from a Dockerfile.

Writing our first Dockerfile

Our Dockerfile must be in a **new, empty directory**.

1.  Create a directory to hold our Dockerfile. $ mkdir myimage

2.  Create a Dockerfile inside this directory.
```
 $ cd myimage

 $ vim Dockerfile
```
Of course, you can use any other editor of your choice, if you are using a Windows device.

Type this into our Dockerfile...
```
FROM ubuntu
RUN apt-get update
RUN apt-get install figlet
```
-   FROM indicates the base image for our build.

-   Each RUN line will be executed by Docker during the build.

-   Our RUN commands **must be non-interactive.**

 (No input can be provided to Docker during the build.)

-   In many cases, we will add the -y flag to apt-get.

 Build it!

Save our file, then execute:
```
 $ docker build -t figlet .
```
-  `-t` indicates the tag to apply to the image.

-   `.` indicates the location of the *build context*.

 (We will talk more about the build context later; but to keep things simple: this is the directory where our Dockerfile is located.)

What happens when we build the image?

The output of docker build looks like this:
```
 $ docker build -t figlet .
 Sending build context to Docker daemon 2.048 kB Sending build context to Docker daemon
 Step 0 : FROM ubuntu
 ---> e54ca5efa2e9
 Step 1 : RUN apt-get update
 ---> Running in 840cb3533193
 ---> 7257c37726a1
 Removing intermediate container 840cb3533193 Step 2 : RUN apt-get install figlet
 ---> Running in 2b44df762a2f
 ---> f9e8f1642759
 Removing intermediate container 2b44df762a2f Successfully built f9e8f1642759
```
-   The output of the RUN commands has been omitted.

-   Let's explain what this output means.

 Sending the build context to Docker

Sending build context to Docker daemon 2.048 kB

 • The build context is the . directory given to docker build.

-   It is sent (as an archive) by the Docker client to the Docker daemon.

-   This allows to use a remote machine to build using local files.

-   Be careful (or patient) if that directory is big and your link is slow.

 Executing each step

Step 1 : RUN apt-get update

---> Running in 840cb3533193 (...output of the RUN command...)

 ---> 7257c37726a1

Removing intermediate container 840cb3533193

-   A container (840cb3533193) is created from the base image.

-   The RUN command is executed in this container.

-   The container is committed into an image (7257c37726a1).

-   The build container (840cb3533193) is removed.

-   The output of this step will be the base image for the next one.

 The caching system

If you run the same build again, it will be instantaneous.

Why?

-   After each build step, Docker takes a snapshot of the resulting image.

-   Before executing a step, Docker checks if it has already built the same sequence.

-   Docker uses the exact strings defined in your Dockerfile, so:

    -   RUN apt-get install figlet cowsay is different from

 RUN apt-get install cowsay figlet

-   RUN apt-get update is not re-executed when the mirrors are updated

You can force a rebuild with docker build --no-cache ....

 Running the image

The resulting image is not different from the one produced manually.

 $ docker run -ti figlet root@91f3c974c9a1:/\# figlet hello

| \_      | \_ \_              |
|---------|--------------------|
| | |\_\_ | \_\_\_| | | \_\_\_ |

 | '\_ \\ / \_ \\ | |/ \_ \\

 | | | | \_\_/ | | (\_) | |\_| |\_|\\\_\_\_|\_|\_|\\\_\_\_/

-   Sweet is the taste of success!


 Using image and viewing history

The history command lists all the layers composing an image.

For each layer, it shows its creation time, size, and creation command.

When an image was built with a Dockerfile, each layer corresponds to a line of the Dockerfile.

$ docker history figlet

| IMAGE           | CREATED             |
|-----------------|---------------------|
| f9e8f1642759    | About an hour ago   |
| 7257c37726a1    | About an hour ago   |
| 07c86167cdc4    | 4                   |
| <missing> | 4                   |
| <missing> | 4                   |
| <missing> | 4                   |

| CREATED BY                       | SIZE    | MB   |     |
|----------------------------------|---------|------|-----|
| /bin/sh -c apt-get install fi    | 1.627   |      |     |
| /bin/sh -c apt-get update        | 21.58   | MB   |     |
| /bin/sh -c \#(nop) CMD \["/bin   | 0 B     | kB   |     |
| /bin/sh -c sed -i 's/^\#\\s\*\\( | 1.895   |      |     |
| /bin/sh -c echo '\#!/bin/sh'     | 194.5   | kB   |     |
| /bin/sh -c \#(nop) ADD file:b    | 187.8   | MB   |     |


Introducing JSON syntax

Most Dockerfile arguments can be passed in two forms:

-   plain string:

 RUN apt-get install figlet

-   JSON list:

 RUN \["apt-get", "install", "figlet"\]

Let's change our Dockerfile as follows!

 FROM ubuntu

 RUN apt-get update

 RUN \["apt-get", "install", "figlet"\]

Then build the new Dockerfile.

 $ docker build -t figlet .

JSON syntax vs string syntax

Compare the new history:

$ docker history figlet

| IMAGE           | CREATED             |
|-----------------|---------------------|
| 27954bb5faaf    | 10 seconds ago      |
| 7257c37726a1    | About an hour ago   |
| 07c86167cdc4    | 4                   |
| <missing> | 4                   |
| <missing> | 4                   |
| <missing> | 4                   |

| CREATED BY                       | SIZE    | MB   |     |
|----------------------------------|---------|------|-----|
| apt-get install figlet           | 1.627   |      |     |
| /bin/sh -c apt-get update        | 21.58   | MB   |     |
| /bin/sh -c \#(nop) CMD \["/bin   | 0 B     | kB   |     |
| /bin/sh -c sed -i 's/^\#\\s\*\\( | 1.895   |      |     |
| /bin/sh -c echo '\#!/bin/sh'     | 194.5   | kB   |     |
| /bin/sh -c \#(nop) ADD file:b    | 187.8   | MB   |     |

-   JSON syntax specifies an *exact* command to execute.

-   String syntax specifies a command to be wrapped within /bin/sh -c "...".

End of Docker labs