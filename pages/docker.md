---
title: Docker
---

# Docker
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Install

Install dnf plugin
```sh
$ sudo dnf -y install dnf-plugins-core
```

Add a repository for Docker CE
```sh
$ sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

Install Docker CE
```sh
$ sudo dnf install docker-ce
```

Start and enable docker daemon
```sh
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

Add a user to the docker group
```sh
$ sudo usermod -aG docker $USER
```

Test `hello-world`
```sh
$ docker run hello-world
```

## Setup Buildx

The buildx is the multi-platform build system using qemu emulation.

Install buildx package
```sh
$ sudo apt-get install -y docker-buildx
```

Install and register qemu
```sh
$ docker run --privileged --rm tonistiigi/binfmt --install all
```

Add following to `/etc/docker/daemon.json` to enable containerd image store on Docker Engine to build multi platform image
```json
{
  "features": {
    "containerd-snapshotter": true
  }
}
```

## Docker Commands

List images in local
```sh
$ docker images
REPOSITORY    TAG        IMAGE ID        CREATED        SIZE
ubuntu        14.04      8cef1fa16c77    2 weeks ago    223MB
hello-world   latest     e38bc07ac18e    5 weeks ago    1.85kB
$ docker image ls
REPOSITORY    TAG        IMAGE ID        CREATED        SIZE
ubuntu        14.04      8cef1fa16c77    2 weeks ago    223MB
hello-world   latest     e38bc07ac18e    5 weeks ago    1.85kB
```

Create a container from a image and run a command
* docker run [options] <image> [command]
  * -i: Keep stdin open (interactive)
  * -t: Allocate a pseudo-tty
  * --rm: Automatically remove the container when it exits
  * -v : Mount a host directory to the container

```sh
$ docker run -it --rm ubuntu:14.04 /bin/bash
root@c82814a1a667:/#

$ docker container run -it --name 'spdk' --hostname spdk -v $HOME/src:/home/workspace -p 80:80 -p 8080:8080 -p 3000:3000 ubuntu:22.04 /bin/bash
```

Execute interactive shell from a docker container which is already executed
```sh
$ docker exec -it <container id> /bin/bash
```

List running containers
* docker ps [options]
  * -a: Show all containers (including stopped containers)

```sh
$ docker ls -a
CONTAINER ID    IMAGE            COMMAND        CREATED            STATUS                    PORTS    NAMES
c82814a1a667    ubuntu:14.04     "/bin/bash"    20 seconds ago     Exited (0) 14 seconds ago          adoring_wescoff
```

Start a stopped container
* docker start <container id>
  * -a: Attach STDOUT/STDERR and forward signals
  * -i: -i: Attach container's STDIN

```sh
$ docker start -ai c82814a1a667
root@c82814a1a667:/#
```

Remove one or more containers
* docker rm <container ids>

```sh
$ docker rm c82814a1a667
c82814a1a667
```

Remove one or more images
* docker rmi <images>

```sh
$ docker rmi hello-world:latest
Untagged: hello-world:latest
Untagged: hello-world@sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Deleted: sha256:e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96
Deleted: sha256:2b8cbd0846c5aeaa7265323e7cf085779eaf244ccbdd982c4931aef9be0d2faf
```

Create a new image from a container
* docker commit [options] <container id> <image>
  * -m: Commit message
  * -a: Author

```sh
$ docker commit -m "commit message" -a "author" c82814a1a667 author/name:tag
```

Build a new image from a Dockerfile
* docker build [options] <path>
  * -t: Name and optionally tag

```sh
$ docker build -t android5-build:ubuntu-14.04 .
```

Export a container's filesystem as a tar archive
* docker export <container> > container.tar

```sh
$ docker export c82814a1a667 > container.tar
```

Save one or more images to a tar archive
* docker save <images> > images.tar

```sh
$ docker save ubuntu:14.04 > image.tar
```

Load an image from a tar archive
```sh
$ docker load < image.tar
```

Clean docker cache
```sh
$ docker system prune -a
```

## Docker Compose Commands

Execute docker
```sh
$ env UID=$(id -u) GID=$(id -g) docker compose -f docker-compose.yml run --rm devel
```

## X11Forwarding

The following options are required to use x11 forwarding in a docker container

Share the Host’s XServer with the Container by creating a volume
```sh
--volume="$HOME/.Xauthority:/root/.Xauthority:rw"
```

Share the Host’s DISPLAY environment variable to the Container
```sh
--env="DISPLAY"
```

Run container with host network driver with
```sh
--net=host
```

## References

* [Docker Docs/Install](https://docs.docker.com/install/linux/docker-ce/fedora/)
* [Docker Docs/Multi-platform builds](https://docs.docker.com/build/building/multi-platform/)
* [Docker Docs/Commandline](https://docs.docker.com/engine/reference/commandline/docker/)
* [Running GUI Applications inside Docker Containers](https://medium.com/@SaravSun/running-gui-applications-inside-docker-containers-83d65c0db110)
