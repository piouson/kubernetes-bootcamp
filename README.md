# CKAD Bootcamp

This course is a part of my cloud-native application developer bootcamp series. Star this repo to say thank you or buy me coffee with the sponsor button.

## Learning Outcomes

In summary, you will be learning cloud-native application development, which is a modern approach to building and running software applications that exploits the flexibility, scalability, and resilience of cloud computing. Some highlights include:

- proficiency working on the command-line
- proficiency working with containers
- proficiency working with infrastructure as code
- microservices architecture
- devops with kubernetes

<details>
  <summary>CKAD exam objectives</summary>

  <a target="_blank" width="100%" align="center" href="https://github.com/cncf/curriculum" alt="CKAD exam curriculum">![image](https://user-images.githubusercontent.com/17856665/186679939-ea79ce57-f277-45c8-8d02-6595d02d9f85.png)</a>
</details>

## Requirements

A Unix-based environment running docker (Docker Engine or Docker Desktop).

- macOS [install Docker Desktop](https://docs.docker.com/desktop/install/mac-install/)
- Windows [setup WSL2 development environment](https://docs.microsoft.com/en-us/windows/wsl/setup/environment), then:
  - option 1 install Docker Engine on WSL2, see below
  - option 2 [install Docker Desktop with WSL2 backend](https://docs.docker.com/desktop/windows/wsl/) - [`winget install Docker.DockerDesktop`](https://docs.microsoft.com/en-us/windows/package-manager/winget/)
- preferably, have a command-line package manager for your operating system
  - macOS use [Homebrew](https://brew.sh/)
  - Windows use [winget](https://docs.microsoft.com/en-us/windows/package-manager/winget/)
  
> Docker Desktop has certain network limitations affecting some network related labs. \
> Consider using Docker Engine if you are **not** on macOS

### Docker engine installation

<details>
  <summary><b>docker engine install steps on Ubuntu/WSL2</b></summary>

```sh
# Windows/WSL2 prerequisites
# a. disable docker desktop integration with WSL2 Ubuntu, see https://docs.microsoft.com/en-us/windows/wsl/media/docker-dashboard.png
# b. enable `systemd` on WSL2
git clone https://github.com/DamionGans/ubuntu-wsl2-systemd-script.git
cd ubuntu-wsl2-systemd-script/
sudo bash ubuntu-wsl2-systemd-script.sh --force
cd ../ && rm -rf ubuntu-wsl2-systemd-script/
exec bash
systemctl # long output confirms systemd up and running

# Docker install steps for Ubuntu/WSL2
# 1. uninstall old docker versions
sudo apt-get remove docker docker-engine docker.io containerd runc
# 2. setup docker repository
sudo apt-get update
sudo apt-get -y install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 3. install docker engine
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
# 4. manage docker as non-root user
sudo groupadd docker
sudo usermod -aG docker $USER
# 5. start a new terminal to update group membership
docker run hello-world
```
</details>

## 1. Understanding and Using Containers

A [container](https://www.docker.com/resources/what-container/) is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.. A **container-runtime**, which relies on the host kernel, is required to run a container.

[Docker](https://www.docker.com/) is most popular container-runtime and container-solution, but there are other runtimes like [runc](https://github.com/opencontainers/runc#runc), [cri-o](https://cri-o.io/), [containerd](https://containerd.io/), etc, but the only significant container-solutions today are Docker and [Podman](https://podman.io/)

A container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Container images become containers at runtime. 

The [Open Container Initiative (OCI)](https://opencontainers.org/) creates open industry standards around container formats and runtimes.

A container registry is a repository, or collection of repositories, used to store and access container images. Container registries are a big player in cloud-native application development, often as part of DevOps processes.

### Container commands

```sh
# run busybox container, see `docker run --help`
docker run busybox
# run in interactive mode
docker run -it busybox
# run in interactive mode and delete container when stopped
docker run -it --rm busybox
# run in detached mode
docker run -d busybox
# run a command in a new container
docker run busybox echo "Hello, World!"
# run container with specified name
docker run -d --name webserver httpd
# use `dash` to execute commands in running container, see `docker exec --help`
docker exec -it $CONTAINER_NAME sh # `sh` will run in a different shell session
# use `bash` to execute commands in running container
docker exec -it $CONTAINER_NAME bash
# list running containers
docker ps
# list all containers
docker ps -a
# view kernel details from inside a container
uname -r
cat /proc/version
hostnamectl
# view os version
cat /etc/redhat-release # for redhat, centos, fedora
cat /etc/os-release # other unix-based os
cat /etc/*-release # works on all popular unix-based os
# view container processes from inside a container
ps aux # view all user-oriented processes, see `ps --help a`
# view container processes, alternative to `ps`
ls /proc # to find PID, then
cat /proc/$PID/cmdline
# exit running container
exit # container is stopped if connected to entrypoint
# exit running container without stopping container
ctrl-p ctrl-q
# start, stop, delete containers, see `docker {start|stop|etc} --help`
docker {start|stop|restart|rm} $CONTAINER_NAME_OR_ID
# see `docker container --help` for complete sub-commands
```

> See [possible container statuses](https://www.baeldung.com/ops/docker-container-states#possible-states-of-a-docker-container) to understand more about container states

### Lab 1.1. Hello docker

1. Run `docker info` to confirm docker client and server statuses
2. Run `docker run hello-world`

### Lab 1.2. First container interaction

1. Run `ps aux` to review running processes on your host device
2. Run a `busybox` container in interactive mode `docker run -it busybox`
3. Review the container kernel details
4. Review the running processes in the container and note PID
5. Exit the container
6. List running containers
7. List all containers
8. Repeat [2] and exit the container without stopping it
9. List running containers
10. List all containers
11. Delete the containers

<details>
  <summary>lab1.2 solution</summary>
  
```sh
# host terminal
ps aux
docker run --name box1 -it busybox
# container terminal
ps aux
uname -r
cat /proc/version
hostnamectl # not found
cat /etc/*-release # not found
busybox | head
exit
# host terminal
docker ps
docker ps -a
docker run --name box2 -it busybox
# container terminal
ctrl+p ctrl+q
# host terminal
docker ps
docker ps -a
docker stop box2
docker rm box1 box2
```
</details>

> `docker ps` showing STATUS of `Exited (0)` means exit OK, but an Exit STATUS that's not 0 should be investigated `docker logs`
>
> `CTRL+P, CTRL+Q` only works when running a container in interactive mode, see [how to attach/detach containers](https://stackoverflow.com/a/19689048/1235675) for more details

### Lab 1.3. Entering and exiting containers

1. Run a `nginx` container
2. List running containers (use another terminal if stuck)
3. Exit the container
4. List running containers
5. Run another `nginx` container in interactive mode
6. Review container kernel details
7. Review running processes in the container
8. Exit container
9. Run another `nginx` container in detached mode
10. List running containers
11. Connect a shell to the new container interactively
12. Exit the container
13. List running containers
14. Delete all containers

<details>
<summary>lab1.3 solution</summary>

```sh
# host terminal
docker run --name webserver1 nginx
# host second terminal
docker ps
# host terminal
ctrl+c
docker ps
docker run --name webserver2 -it --rm nginx bash
# container terminal
cat /etc/*-release
ps aux # not found
ls /proc
ls /proc/1 # list processes running on PID 1
cat /proc/1/$PROCESS_NAME
exit
# host terminal
docker run --name webserver3 -d nginx
docker ps
docker exec -it webserver3 bash
# container terminal
exit
# host terminal
docker ps
docker stop webserver3
docker rm webserver1 webserver2 webserver3
```
</details>

> Containers may not usually have `bash` shell, but will usually have the dash shell `sh`

### Lab 1.4. Container arguments

1. Run a `busybox` container with command `sleep 30` as argument, see `sleep --help`
2. List running containers (use another terminal if stuck)
3. Exit container (note that container will auto exit after 30s)
4. Run another `busybox` container in detached mode with command `sleep 300` as argument
5. List running containers
6. Connect to the container to execute commands
7. Exit container
8. List running containers
9. Run another `busybox` container in detached mode, no commands
10. List running containers
11. List all containers
12. Delete all containers

<details>
<summary>lab1.4 solution</summary>

```sh
# host terminal
docker run --name box1 busybox sleep 30
# host second terminal
docker ps
docker stop box1
# host terminal
docker run --name box2 -d busybox sleep 300
docker ps
docker exec -it box2 sh
# container terminal
exit
# host terminal
docker ps
docker run --name box3 -d busybox
docker ps
docker ps -a
docker stop box2
docker rm box1 box2 box3
```
</details>

> The `Entrypoint` of a container is [the init process](https://en.wikipedia.org/wiki/Init) and allows the container to run as an executable. Commands passed to a container are passed to the container's entrypoint process.
>
> Note that `docker` commands after `imageName` are passed to the container's entrypoint as arguments. \
> ❌ `docker run -it mysql -e MYSQL_PASSWORD=hello` will pass `-e MYSQL_PASSWORD=hello` to the container
> ✔️ `docker run -it -e MYSQL_PASSWORD=hello mysql`

### Managing containers and images

```sh
# run container with port, see `docker run --help`
docker run -d -p 8080:80 httpd # visit localhost:8080
# run container with mounted volume
docker run -d -p 8080:80 -v ~/html:/usr/local/apache2/htdocs httpd
# run container with environment variable
docker run -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=secret mongo
# inspect container, see `docker container inspect --help | docker inspect --help`
docker inspect $CONTAINER_NAME_OR_ID | less
docker container inspect $CONTAINER_NAME_OR_ID
# format inspect output to view container network information
docker inspect --format="{{.NetworkSettings.IPAddress}}" $CONTAINER_NAME_OR_ID
# format inspect output to view container status information
docker inspect --format="{{.State.Pid}}" $CONTAINER_NAME_OR_ID
# view container logs, see `docker logs --help`
docker logs $CONTAINER_NAME_OR_ID
# remove all unused data (including dangling images)
docker system prune
# remove all unused data (including unused images, dangling or not, and volumes)
docker system prune --all --volumes
# manage images, see `docker image --help`
docker image ls # or `docker images`
docker image inspect $IMAGE_ID
docker image rm $IMAGE_ID
# see `docker --help` for complete resources
```

### Lab 1.5. Container ports and IP

1. Run a `nginx` container with name `webserver`
2. Inspect the container (use `| less` to avoid console clutter) and review the `State` and `NetworkSettings` fields, quit with `q`
3. Visit `http://$CONTAINER_IP_ADDRESS` in your browser (this may not work depending on your envrionment network settings)
4. Run another `nginx` container with name `webserver` and exposed on port 80
5. Visit http://localhost in your browser
6. Delete the containers

<details>
<summary>lab1.5 solution</summary>

```sh
# host terminal
docker run -d --name webserver nginx
docker inspect webserver | grep -A 13 '"State"' | less
docker inspect webserver | grep -A 50 '"NetworkSettings"' | less
curl http://$(docker inspect webserver --format "{{.NetworkSettings.IPAddress}}") | less
docker stop webserver
docker rm webserver
docker run -d --name webserver -p 80:80 nginx
curl localhost | less
docker ps
docker ps -a
docker stop webserver
docker rm webserver
```
</details>

> Always run containers in detached mode to avoid getting stuck in the container `STDOUT`

### Lab 1.6. Container volumes

1. Create an `html/index.html` file with some content
2. Run any webserver containers on port 8080 and mount the `html` folder to the [DocumentRoot](https://serverfault.com/a/588388)
   - option `nginx` DocumentRoot - `/usr/share/nginx/html`
   - option `httpd` DocumentRoot - `/usr/local/apache2/htdocs`
3. Visit http://localhost:8080
4. List running containers
5. List all containers
6. Delete containers

<details>
<summary>lab1.6 solution</summary>

```sh
# host terminal
cd ~
mkdir html
echo "Welcome to Lab 1.6 Container volumes" >> html/index.html
# with nginx
docker run -d --name webserver -v ~/html:/usr/share/nginx/html -p 8080:80 nginx
# with httpd
# docker run -d --name webserver -v ~/html:/usr/local/apache2/htdocs -p 8080:80 httpd
curl localhost:8080
docker ps
docker ps -a
docker stop webserver
docker rm webserver
```
</details>

### Lab 1.7. Container environment variables

1. Run a `mysql` container in detached mode
2. Connect to the container
3. Review the container logs and resolve the output message regarding environment variable
4. Confirm issue resolved by connecting to the container
5. Exit the container
6. List running containers
7. List all containers
8. List all images
9. List all volumes
10. Clean up with [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)
11. Check all resources are deleted, containers, images and volumes.

<details>
<summary>lab1.7 solution</summary>

```sh
# host terminal
docker run -d --name db mysql
docker exec -it db bash # error not running
docker logs db
docker rm db
docker run -d --name db -e MYSQL_ROOT_PASSWORD=secret mysql
docker ps
docker ps -a
docker image ls
docker volume ls
docker stop db
docker ps # no containers running
docker system prune --all --volumes
docker image ls
docker volume ls
```
</details>

> You don't always have to run a new container, we have had to do this to apply new configuration. You can restart an existing container `docker ps -a`, if it meets your needs, with `docker start $CONTAINER`

### Lab 1.8. Container registries

Explore [Docker Hub](https://hub.docker.com/) and search for images you've used so far or images/applications you use day-to-day, like databases, environment tools, etc.

> Container images are created with instructions that determine the default container behaviour at runtime. A familiarity with specific images/applications may be required to understand their default behaviours

## 2. Managing Container Images

A docker image consist of layers, and each [image layer](https://vsupalov.com/docker-image-layers/) is its own image. An image layer is a change on an image - every command (FROM, RUN, COPY, etc.) in your Dockerfile (aka Containerfile by OCI) causes a change, thus creating a new layer. It is recommended reduce your image layers as best possible, e.g. replace multiple `RUN` commands with "command chaining" `apt update && apt upgrade -y`.

> A name can be assigned to an image by "tagging" the image. This is often used to identify the image version and/or registry.

```sh
# to view image layers/history, see `docker image history --help`
docker image history $IMAGE_ID
# tagging images, see `docker tag --help`
docker tag $IMAGE_NAME $NEW_NAME:$TAG # if tag is omitted, `latest` is used
docker tag nginx nginx:1.1
# tags can also be used to add repository location
docker tag nginx domain.com/nginx:1.1
```

### Lab 2.1. Working with images

1. List all images (if you've just finished [lab1.7](#lab-17-container-environment-variables), run new container to download an image)
2. Inspect one of the images with `| less` and review the `ContainerConfig` and `Config`
3. View the image history
4. Tag the image with the repository `localhost` and a version
5. List all images
6. View the tagged image history
7. Delete tagged image by ID
8. Lets try that again, delete tagged image by tag

<details>
<summary>lab2.1 solution</summary>

```sh
# host terminal
docker image ls
# using nginx image
docker image inspect nginx | grep -A 40 ContainerConfig | less
docker image inspect nginx | grep -A 40 '"Config"' | less
docker image history nginx
docker tag nginx localhost/nginx:1.1
docker image ls
docker image history localhost/nginx:1.1 # tagging isn't a change
docker image rm $IMAGE_ID # error conflict
docker image rm localhost/nginx:1.1 # deleting removes tag
```
</details>

### Lab 2.2. Custom images

Although, we can also [create an image from a running container using `docker commit`](https://docs.docker.com/engine/reference/commandline/commit/), we will only focus on using a Dockerfile, which is the recommended method.

Build the below Dockerfile with `docker build -t $IMAGE_NAME:$TAG /path/to/Dockerfile/directory`, see `docker build --help

```Dockerfile
# Example Dockerfile
FROM ubuntu
MAINTAINER Piouson
RUN apt-get update && \
    apt-get install -y nmap iproute2 && \
    apt-get clean
ENTRYPOINT ["/usr/bin/nmap"]
CMD ["-sn", "172.17.0.0/16"] # nmap will scan docker network subnet `172.17.0.0/16` for running containers
```

### Dockerfile overview

```sh
FROM # specify base image
RUN # execute commands
ENV # specify environment variables used by container
ADD # copy files from project directory to the image
COPY # copy files from local project directory to the image - ADD is recommended
ADD /path/to/local/file /path/to/container/directory # specify commands in shell form - space separated
ADD ["/path/to/local/file", "/path/to/container/directory"] # specify commands in exec form - as array (recommended)
USER # specify username (or UID) for RUN, CMD and ENTRYPOINT commands
ENTRYPOINT ["command"] # specify default command, `/bin/sh -c` is used if not specified - cannot be overwritten, so CMD is recommended for flexibility
CMD ["arg1", "arg2"] # specfify arguments to the ENTRYPOINT - if ENTRYPOINT is not specified, args will be passed to `/bin/sh -c`
```

### Lab 2.2. Create image from Dockerfile

See [best practices for writing Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices).

```sh
# find a package containing an app (debian-based)
apt-file search --regex <filepath-pattern> # requires `apt-file` installation, see `apt-file --help`
apt-file search --regex ".*/sshd$"
# find a package containing an app, if app already installed (debian-based)
dpkg -S /path/to/file/or/pattern # see `dpkg --help`
dpkg -S */$APP_NAME
# find a package containing an app (rpm-based)
dnf provides /path/to/file/or/pattern
dnf provides */sshd
```

1. Create a Dockerfile based on the following:
   - Base image should be debian-based or rpm-based
   - Should include packages containing `ps` application and network utilities like `ip`, `ss` and `arp`
   - Should run the `nmap` process as the `ENTRYPOINT` with arguments `-sn 172.17.0.0/16`
2. Build the Dockerfile with repository `local` and version `1.0`
3. List images
4. Run separate containers from the image as follows and review behaviour
   - do not specify any modes
   - in interactive mode with a shell
   - in detached mode, then check the logs
5. Edit the Dockerfile to run the same process and arguments but **not** as `ENTRYPOINT`
6. Repeat all three options in [4] and compare the behaviour
7. Clean up

<details>
<summary>lab2.2 solution</summary>

```sh
# run ubuntu container to find debian-based packages
docker run -it --rm ubuntu
# container terminal
apt update
apt install -y apt-file
apt-file update
apt-file search --regex "bin/ip$"
apt-file search --regex "bin/ss$"
apt-file search --regex "bin/arp$"
# found `iproute2` and `net-tools`
exit
```

```sh
# alternatively, run fedora container to find rpm-based packages
docker run -it --rm fedora
# container terminal
dnf provides *bin/ip
dnf provides *bin/ss
dnf provides *bin/arp
# found `iproute` and `net-tools`
exit
```

```sh
# host terminal
mkdir test
nano test/Dockerfile
```

```Dockerfile
# Dockerfile
FROM alpine
RUN apk add --no-cache nmap iproute2 net-tools
ENTRYPOINT ["/usr/bin/nmap"]
CMD ["-sn", "172.17.0.0/16"]
```

```sh
# host terminal
docker build -t local/alpine:1.0 ./test
docker run --name alps1 local/alpine:1.0
docker run --name alps2 -it local/alpine:1.0 sh
docker run --name alps3 -d local/alpine:1.0
docker log alps3
nano test/Dockerfile
```

```Dockerfile
# Dockerfile
FROM alpine
RUN apk add --no-cache nmap iproute2 net-tools
CMD ["/usr/bin/nmap", "-sn", "172.17.0.0/16"]
```

```sh
# host terminal
docker build -t local/alpine:1.1 ./test
docker run --name alps4 local/alpine:1.0
docker run --name alps5 -it local/alpine:1.0 sh
# container terminal
exit
# host terminal
docker run --name alps6 -d local/alpine:1.0
docker log alps6
docker stop alps3 alps5 alps6
docker rm alps1 alps2 alps3 alps4 alps5 alps6
docker image rm local/alpine:1.0 local/alpine:1.1
```
</details>

> In most cases, building an image goes beyond a successful build. Some installed packages require additional steps to run containers successfully

### Lab 2.3. Containerise your application

See the [official language-specific getting started guides](https://docs.docker.com/language/) which includes NodeJS, Python, Java and Go examples.

1. Bootstrap a frontend/backend application project, your choice of language
2. Install all dependencies and test the app works
3. Create a Dockerfile to containerise the project
4. Build the Dockerfile
5. Run a container from the image exposed on port 8080
6. Confirm you can access the app on http://localhost:8080

<details>
<summary>lab2.3 nodejs solution</summary>

```sh
# host terminal
npx express-generator --no-view test-app
cd test-app
yarn
yarn start # visit localhost:3000 if OK, ctrl+c to exit
echo node_modules > .dockerignore
nano Dockerfile
```

```Dockerfile
# Dockerfile
FROM node:alpine
ENV NODE_ENV=production
WORKDIR /app
COPY ["package.json", "yarn.lock", "./"]
RUN yarn --frozen-lockfile --prod
COPY . .
CMD ["node", "bin/www"]
EXPOSE 3000
```

```sh
# host terminal
docker build -t local/app:1.0 .
docker run -d --name app -p 8080:3000 local/app:1.0
curl localhost:8080
docker stop app
docker rm app
docker image rm local/app:1.0
cd ..
rm -rf test-app
```
</details>

### Docker container access control

Before we finally go into Kubernetes, it would be advantageous to have a basic understanding of unix-based systems file permissions and access control.

#### UID and GID

A _user identifier (UID)_ is a unique number assigned to each user. This is how the system identifies each user. The root user has UID of 0, UID 1-500 are often reserved for system users and UID for new users commonly start at 1000. UIDs are stored in the plain-text `/etc/passwd` file: each line represents a user account, and has seven fields delimited by colons `account:password:UID:GID:GECOS:directory:shell`.

A _group identifier (GID)_ is similar to UIDs - used by the system to identify _groups_. A _group_ consists of several users and the root group has GID of 0. GIDs are stored in the plain-text `/etc/group` file: each line represents a group, and has four fields delimited by colons `group:password:GID:comma-separated-list-of-members`. An example of creating and assigning a group was covered in [step 4 - docker engine installation](#docker-engine-installation) where we created and assigned the `docker` group.

UIDs and GIDs are used to implement _Discretionary Access Control (DAC)_ in unix-based systems by assigning them to files and processes to denote ownership - _left at owner's discretion_. This can be seen by running `ls -l` or `ls -ln`: the output has seven fields delimited by spaces `file_permisions number_of_links user group size date_time_created file_or_folder_name`. See [unix file permissions](https://www.guru99.com/file-permissions.html) for more details.

<details>
<summary><code>ls -l</code> in detail</summary>

![output of commandline list](https://user-images.githubusercontent.com/17856665/187016629-e6c4f17f-f06a-4aeb-98c4-0cfc4d371be2.png)
</details>

```sh
# show current user
whoami
# view my UID and GID, and my group memberships
id
# view the local user database on system
cat /etc/passwd
# output - `account:password:UID:GID:GECOS:directory:shell`
root:x:0:0:root:/root:/bin/bash
piouson:x:1000:1000:,,,:/home/dev:/bin/bash
# view the local group database on system
cat /etc/group
# output - `group:password:GID:comma-separated-list-of-member`
root:x:0:
piouson:x:1000:
docker:x:1001:piouson
# list folder contents and their owner (user/group) names 
ls -l 
# show ownership by ids, output - `permision number_of_links user group size date_time_created file_or_folder_name`
ls -ln
```

#### Capabilities

In the context of permission checks, processes running on unix-based systems are traditionally categorised as:

- _privileged_ processes: effective UID is 0 (root) - bypass all kernel permission checks
- _unprivileged_ processes: effective UID is nonzero - subject to permission checks

Starting with kernel 2.2, Linux further divides traditional root privileges into distinct units known as _capabilities_ as a way to control root user powers. Each root _capability_ can be independently enabled and disabled, and uses a naming convention prefix `CAP_`. \
See the [overview of Linux _capabilities_](https://man7.org/linux/man-pages/man7/capabilities.7.html) for more details, including a comprehensive list of capabilities.

> `CAP_SYS_ADMIN` is an overloaded capability that grants privileges similar to traditional root privileges \
> By default, Docker containers are **_unprivileged_** and root in a docker container uses [_restricted capabilities_](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) \
> ❌ `docker run --privileged` gives all capabilities to the container, allowing nearly all the same access to the host as processes running on the host

#### Running containers as non-root

For practical reasons, most containers run as root by default. However, in a security context, this is bad practice:

- it voilates the [_principle of least privilege_](https://en.wikipedia.org/wiki/Principle_of_least_privilege)
- an attacker might take advantage of an application vulnerability to gain root access to the container
- an attacker might take advantage of a container-runtime, or kernel, vulnerability to gain root access to the host after gaining access to the container

We can control the users containers run with by:

- omitting the `USER` command in Dockerfile assigns root
- specify a user in the `Dockerfile` with the `USER` command
- override the UID at runtime with `docker run --user $UID`

```Dockerfile
# Dockerfile
FROM ubuntu
# create group `piouson`, and create user `piouson` as member of group `piouson`, see `groupadd -h` and `useradd -h`
RUN groupadd piouson && useradd piouson --gid piouson
# specify GID/UID when creating/assigning a group/user
RUN groupadd --gid 1004 piouson && useradd --uid 1004 piouson --gid piouson
# assign user `piouson` for subsequent commands
USER piouson
# create system-group `myapp`, and create system-user `myapp` as member of group `myapp`
RUN groupadd --system myapp && useradd --system --no-log-init myapp --gid myapp
# assign system-user `myapp` for subsequent commands
USER myapp
```

### Lab 2.4. Container privileges

1. Display your system's current user
2. Display the current user's UID, GID and group memberships
3. Run a `ubuntu` container interactively, and in the container shell:
   - display the current user
   - display the current user's UID, GID and group memberships
   - list existing user accounts
   - list existing groups
   - create a file called `test-file` and display the file ownership info
   - exit the container
4. Run a new `ubuntu` container interactively with UID 1004, and in the container shell:
   - display the current user
   - display the current user's UID, GID and group memberships
   - exit the container
5. Create a docker image based on `ubuntu` with a non-root user as default user
6. Run a container interactively using the image, and in the container shell:
   - display the current user
   - display the current user's UID, GID and group memberships
   - exit the container
7. Delete created resources

<details>
<summary>lab2.4 solution</summary>

```sh
# host terminal
whoami
id
docker run -it --rm ubuntu
# container terminal
whoami
id
cat /etc/passwd
cat /etc/group
touch test-file
ls -l
ls -ln
exit
# host terminal
docker run -it --rm --user 1004 ubuntu
# container terminal
whoami
id
exit
```

```Dockerfile
# test/Dockerfile
FROM ubuntu
RUN groupadd --gid 1000 piouson && useradd --uid 1000 piouson --gid 1000
USER piouson
```

```sh
# host terminal
docker build -t test-image test/
docker run -it --rm test-image
# container terminal
whoami
id
exit
# host terminal
docker image rm test-image
```
</details>

> If a containerized application can run without privileges, change to a non-root user \
> It is recommended to explicitly specify GID/UID when creating a group/user

## 3. Understanding Kubernetes

[K8s](kubernetes.io) is an open-source system for automating deployment, scaling and containerized applications management, currently owned by the [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/). \
K8s **release cycle is 3 months** and deprecated features are supported for a minimum of 2 release cycles (6 months).

> You can watch [kubernetes in 1 minute](https://www.youtube.com/watch?v=BzvLp-lH5_Q&list=PLBBog2r6uMCSplEmHu-1n7VixRn9RTZP5&index=6) for a quick overview \
> When you've got more time, watch/listen to **Kubernetes: The Documentary ([PART 1](https://www.youtube.com/watch?v=BE77h7dmoQU) & [PART 2](https://www.youtube.com/watch?v=318elIq37PE))**

### Lab 3.1. Kubernetes in Google Cloud

> A local lab setup is covered in [chapter 4 with minikube](#4-kubernetes-lab-environment) \
> Skip this lab if you do not currently have a Google Cloud account with Billing enabled

- Signup and Login to [console.cloud.google.com](https://console.cloud.google.com)
- Use the "Cluster setup guide" to create "My first cluster"
- Connect to the cluster using the "Cloud Shell"
- View existing Kubernetes resources by running `kubectl get all`

### Basic Kubernetes API resources

Entities in Kubernetes are recorded in the Kubernetes system as _Objects_, and they represent the state of your cluster. [_Kubernetes objects_](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects) can describe:

- what containerized applications are running (and on which nodes)
- resources available to those applications
- policies around applications behaviour - restarts, upgrades, fault-tolerance, etc

Some common _Kubernetes objects_ include:

- Deployment: represents the application and provides services
- ReplicaSet: manages scalability - array of pods
- Pods: manages containers (**note that one container per pod is the standard**)

<details>
  <summary>Kubernetes architecture</summary>
  
![kubernetes-architecture from https://platform9.com/blog/kubernetes-enterprise-chapter-2-kubernetes-architecture-concepts/](https://user-images.githubusercontent.com/17856665/185714430-afe68ed2-9593-47c1-b032-b9ad9630f9fa.png)
</details>

<details>
  <summary>Pods architecture</summary>
  
![pods-architecture from https://platform9.com/blog/kubernetes-enterprise-chapter-2-kubernetes-architecture-concepts/](https://user-images.githubusercontent.com/17856665/185716058-b9b273ab-295b-4a5a-9d63-31b6f0d25e2c.jpg)
</details>

```sh
# help
kubectl --help | less
# view available resources
kubectl get all, see `kubectl get --help`
# create a deployment, see `kubectl create deploy -h`
kubectl create deploy myapp --image=nginx
# create a deployment with six replicas
kubectl create deploy myapp --image=nginx --replcias=6
# view complete list of supported API resources
kubectl api-resources
# delete a deployment, see `kubectl delete --help`
kubectl delete deploy myapp
```

### Lab 3.2. Explore Kubernetes API resources via Google Cloud

> This lab is repeated in [chapter 4 with minikube](#4-kubernetes-lab-environment) \
> Skip this lab if you do not currently have a Google Cloud account with Billing enabled

1. Create an `nginx` application with three replicas
2. View available resources
3. Delete a pod create
4. View available resources, how many pods left, can you find the deleted pod?
5. List supported API resources
6. Delete the application
7. view available resource
8. Delete the Kubernetes service
9. view available resources
10. If nothing found, allow 5s and try [9] again

<details>
<summary>lab3.2 solution</summary>

```sh
kubectl create deploy webserver --image=nginx --replicas=3
kubectl get all
kubectl delete pod $POD_NAME
kubectl get all # new pod auto created to replace deleted
kubectl api-resources
kubectl delete deploy webserver
kubectl get all
kubectl delete svc kubernetes
kubectl get all # new kubernetes service is auto created to replace deleted
```
</details>

> Remember to delete Google cloud cluster to avoid charges if you wish to use a local environment detailed in the next chapter

## 4. Kubernetes Lab Environment

```sh
# check kubernetes version
kubectl version
# list kubernetes context (available kubernetes clusters - docker-desktop, minikube, etc)
kubectl config get-contexts
# switch kubernetes context
kubectl config use-context docker-desktop
```

### Use Docker Desktop

<details>
  <summary>Enable Kubernetes in Docker Desktop settings, then apply and restart</summary>
  
  ![image](https://user-images.githubusercontent.com/17856665/178120815-357cea98-ecf1-4536-81af-a614b7b4cf5e.png)
</details>

> See Docker's [Deploy on Kubernetes](https://docs.docker.com/desktop/kubernetes/) for more details \
> Note that using Docker Desktop will have network limitations when exposing your applications publicly, see alternative Minikube option below

### Use Minikube

Minikube is the recommended Kubernetes solution for this course on a local lab environment. See the [official minikube installation docs](https://minikube.sigs.k8s.io/docs/start/).

> [Kubernetes v1.24+ has deprecated docker as a container-runtime and now uses cri-o](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/). Therefore, we use Kubernetes v1.23 to avoid installing cri-o and its dependencies.

#### Install Minikube on macOS

```sh
# 1. install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
rm minikube-darwin-amd64
# 2. start a minikube cluster
minikube start --kubernetes-version=1.23.9
```

#### Install Minikube on Windows WSL2

```sh
# 1. install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x ./minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
# 2. install minikube prereqs - conntrack
sudo apt install conntrack
sudo sysctl fs.protected_regular=0
# 3. start a minikube cluster
minikube start --driver=docker --kubernetes-version=1.23.9
# 3b. optional, start a minikube cluster with `--driver=none`
sudo minikube start --driver=none --kubernetes-version=1.23.9
# 4. change the owner of the .kube and .minikube directories
sudo chown -R $USER $HOME/.kube $HOME/.minikube
```

#### Minikube commands

```sh
# show current status, see `minikube --help`
minikube status
# open K8s dashboard in local browser
minikube dashboard
# start, stop, delete cluster, see `minikube {start|stop|delete} --help`
minikube {start|stop|delete}
# start minikube with a specified driver and specified kubernetes version
minikube start --driver=docker --kubernetes-version=1.23
# show current IP address
minikube ip
# show current version
minikube version
# connect to minikube cluster
minikube ssh
# list addons
minikube addons list
# enable addons
minikube addons enable [addonName]
```

### Lab 4.1. Setup minikube and kubectl

1. Confirm minikube running `minikube status`
2. Create `kubectl` alias in `.bashrc`
   ```sh
   printf "
   # minikube kubectl
   alias kubectl='minikube kubectl --'
   " >> ~/.bashrc
   exec bash
   ```
3. Start using the alias
   ```sh
   kubectl version
   kubectl get all
   ```
4. Enable kubectl autocompletion, see `kubectl completion --help`
   ```sh
   echo "source <(kubectl completion bash)" >> ~/.bashrc # macos replace bash with zsh
   exec bash
   ```
5. The default `kubectl edit` text editor is `vi`. To change this:
   ```sh
   export KUBE_EDITOR="nano" # use nano
   export KUBE_EDITOR="vim" # use vim
   ```
6. Open the kubernetes dashboard with `minikube dashboard`
7. Use the Kubernetes Dashboard to deploy a webserver with three replicas
   - visit url provided in browser
   - click on top right plus "+" icon
   - select `Create from form`
   - enter App name: `app`, Container image: `nginx`, Number of pods: `3`
   - click `Deploy`
8. Return to the terminal and delete created resources
   ```sh
   ctrl+c # to terminate dashboard
   kubectl get all
   kubectl delete deploy app
   ```

### Lab 4.2. Explore Kubernetes API resources via Minikube

1. Run an `nginx` Pod
2. View resources
3. Delete the Pod
4. View resources
5. Repeat [Lab 3.2](#lab-32-explore-kubernetes-api-resources-via-gcloud) in Minikube

<details>
<summary>lab4.2 solution</summary>

```sh
kubectl run webserver --image=nginx
kubectl get all
kubectl delete pod webserver
kubectl get all # pod gone
# see `lab3.2 solution` for remaining steps
```
</details>

> Pods started without a deployment are called _Naked Pods_ - these are not managed by a replicaset, therefore, are not rescheduled on failure, not eligible for rolling updates, cannot be scaled, cannot be replaced automatically. \
Although, _Naked Pods_ are not recommended in live environments, they are crucial for learning how to manage Pods, which is a big part of CKAD.

## 5. Pods

[Pods](https://kubernetes.io/docs/concepts/workloads/pods/) are the smallest deployable units of computing that you can create and manage in Kubernetes.

###  Managing Pods

```sh
# run a pod, see `kubectl run --help`
kubectl run $POD_NAME $IMAGE_NAME
# run a nginx pod with custom args, args are passed to the pod's container's `ENTRYPOINT`
kubectl run mypod --image=nginx -- <arg1> <arg2> ... <argN>
# run a command in an nginx pod
kubectl run mypod --image=nginx --command -- <command>
# run a busybox pod interactively and delete after task completion
kubectl run -it mypod --image=busybox --rm --restart=Never -- date
# display running pods, see `kubectl get --help`
kubectl get pods # using `pod` or `pods` will work
# display full details of pod in YAML form
kubectl get pods $POD_NAME -o yaml | less
# show details of pod in readable form, see `kubectl describe --help`
kubectl describe pods $POD_NAME | less
```

> With `kubectl`, everything after the ` -- ` flag is passed to the Pod \
> 💡 `-- <args>` corresponds to Dockerfile `CMD` while `--command -- <args>` corresponds to `ENTRYPOINT` \
> See [answer to `kubectl run --command vs -- arguments`](https://stackoverflow.com/a/66078726) for more details

### Lab 5.1. Creating Pods

1. Create an nginx Pod and confirm creation
2. Review full details of the Pod in YAML form
3. Display details of the Pod in readable form and review the Node, IP, container start date/time and Events
4. Delete the Pod

<details>
<summary>lab5.1 solution</summary>

```sh
kubectl run mypod --image=nginx
kubectl get pods
kubectl describe pods mypod | less
kubectl delete pods mypod
```
</details>

### Pod manifest file

Example of a Pod manifest file with a `busybox` image and mounted _empty-directory_ volume.

```yaml
apiVersion: v1 # api version
kind: Pod # type of resource, pod, deployment, configmap, etc
metadata:
  name: box # metadata information, including labels, namespace, etc
spec:
  containers:
  - name: box
    image: busybox:1.28
    volumeMounts: # mount created volume
    - name: varlog
      mountPath: /var/log
  volumes: # create an empty-directory volume
  - name: varlog
    emptyDir: {}
```

> Volumes are covered in more detail in [Chapter 10 - Storage](#10-storage). For now it will suffice to know how to create and mount an _empty-directory_ volume

```sh
# view fields description of a Kubernetes Object - see `kubectl explain --help`
kubectl explain <object>[.field]
kubectl explain pod
kubectl explain pod.metadata # or `pod.spec`, `pod.status` etc
# include nested fields with `--recursive`
kubectl explain --recursive pod.spec | less
# perform actions on a resource with a YAML file
kubectl {create|apply|replace|delete} -f pod.yaml
# generate YAML file of a specific command with `--dry-run`
kubectl run mynginx --image=nginx -o yaml --dry-run=client > pod.yaml
```

> Object fields are **case sensitive**, **always generate** manifest files to avoid typos \
> `kubectl apply` creates a new resource, or updates existing if previously created by `kubectl apply`

### Valid reasons for multi-container Pods

**Always create single container Pods!** However, some special scenarios require a [multi-container Pod pattern](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/):

- To initialise primary container ([Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#understanding-init-containers))
- To enhance primary container, e.g. for logging, monitoring, etc. ([Sidecar Container](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-1-sidecar-containers))
- To prevent direct access to primary container, e.g. proxy ([Ambassador Container](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-2-ambassador-containers))
- To match the traffic/data pattern in other applications in the cluster ([Adapter Container](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-3-adapter-containers))

### Lab 5.2. Managing Pods with YAML file

1. Generate a YAML file of a `busybox` Pod that runs the command `sleep 60`, see [create Pod with command and args docs](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#define-a-command-and-arguments-when-you-create-a-pod)
2. Apply the YAML file.
3. List created resources
4. View details of the Pod.
5. Delete the Pod

<details>
<summary>lab5.2 solution</summary>

```sh
kubectl run mypod --image=busybox --dry-run=client -o yaml --command -- sleep 60 > lab5-2.yaml
kubectl apply -f lab5-2.yaml
kubectl get pods
kubectl describe pods myapp | less
kubectl delete -f lab5-2.yaml
```
</details>

> Practice managing resources mainly with YAML file for all Labs going forward

### Lab 5.3. Init Containers

Note that the main container will only be started after the _init container_ enters `STATUS=completed`

```sh
# view logs of pod `mypod`
kubectl logs mypod
# view logs of specific container `mypod-container-1` in pod `mypod`
kubectl logs mypod -c mypod-container-1
```

1. Create a Pod that logs `App is running!` to STDOUT
   - the application should `Never` restart
   - the application should use a _Init Container_ to wait for 60secs before starting
   - the _Init Container_ should log `App is initialising...` to STDOUT
   - see [init container docs](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use).
2. List created resources and note Pod `STATUS`
3. View the logs of the main container
4. View the logs of the _init container_
5. View more details of the Pod and note the `State` of both containers.
6. List created resources and confirm Pod `STATUS`
7. Delete Pod

<details>
<summary>lab5.3 solution</summary>

```sh
# partially generate pod manifest
kubectl run myapp --image=busybox --restart=Never --dry-run=client -o yaml --command -- sh -c "echo App is running!" > lab5-3.yaml
```

```yaml
# edit lab5-3.yaml to add init container spec
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: myapp
  name: myapp
spec:
  containers:
  - name: myapp
    image: busybox:1.28
    command: ['sh', '-c', 'echo App is running!']
  initContainers:
  - name: myapp-init
    image: busybox:1.28
    command: ['sh', '-c', 'echo "App is initialising..." && sleep 60']
  restartPolicy: Never
```

```sh
kubectl apply -f lab5-3.yaml
kubectl get pods
kubectl logs myapp # not created until after 60secs
kubectl logs myapp -c myapp-logs
kubectl describe -f lab5-3.yaml | less
kubectl get pods
kubectl delete -f lab5-3.yaml
```
</details>

### Lab 5.4. Multi-container Pod

1. Create a Pod with 2 containers and a volumne shared by both containers, see [multi-container docs](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/#creating-a-pod-that-runs-two-containers).
2. List created resources
3. View details of the Pod.
5. Delete the Pod

<details>
<summary>lab5.4 solution</summary>

```yaml
# lab5-4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp-1
    image: busybox:1.28  
    volumeMounts:
    - name: logs
      mountPath: /var/log
  - name: myapp-2
    image: busybox:1.28
    volumeMounts:
    - name: logs
      mountPath: /var/log
  volumes:
  - name: logs
    emptyDir: {}
```

```sh
kubectl apply -f lab5-4.yaml
kubectl get pods
kubectl describe pods myapp | less
kubectl logs myapp
kubectl logs myapp -c myapp-logs
kubectl delete -f lab5-4.yaml
```
</details>

> Always create single container Pods!

### Lab 5.5. Sidecar Containers

```sh
# download a file
wget https://url/of/file.extension
# save downloaded file with a new name
wget https://url/of/file.extension -O new-name.extension
# hide output while downloading
wget -q https://url/of/file.extension
# view contents of a downloaded file without saving, use `-q -O-` for quiet mode
wget -O- https://url/of/file.extension
```

> Note that you can prepend [`https://k8s.io/examples/`](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods) to any example files in the official docs for direct download of the YAML file

1. Create a `busybox` Pod that logs `date` to a file every second
   - expose the logs with a _sidecar container_'s STDOUT to prevent direct access to the main application
   - see example _sidecar container_ manifest `https://k8s.io/examples/admin/logging/two-files-counter-pod-streaming-sidecar.yaml`
2. List created resources
3. View details of the Pod
4. View the logs of the main container
5. View the logs of the _sidecar container_
6. Delete created resources

<details>
<summary>lab5.5 solution</summary>

```yaml
# lab5-5.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      while true;
      do
        echo $(date) >> /var/log/date.log;
        sleep 1;
      done      
    volumeMounts:
    - name: logs
      mountPath: /var/log
  - name: myapp-logs
    image: busybox:1.28
    args: [/bin/sh, -c, 'tail -F /var/log/date.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log
  volumes:
  - name: logs
    emptyDir: {}
```

```sh
kubectl apply -f lab5-5.yaml
kubectl get pods
kubectl describe pods myapp | less
kubectl logs myapp
kubectl logs myapp -c myapp-logs
kubectl delete -f lab5-5.yaml
```
</details>

### Using namespaces

```sh
# create namespace called `myns`
kubectl create namespace myns
# run a pod in the `myns` namespace with `-n myns`
kubectl run mypod --image=imageName -n namespaceName
# view pods in the `myns` namespaces
kubectl get pods -n myns
# view pods in all namespaces with `--all-namespaces` or `-A`
kubectl get pods --all-namespaces
# view all resources in all namespaces
kubectl get all --all-namespaces
# view the current namespace in use for commands
kubectl config view --minify | grep namespace:
# set `myns` namespace to be the namespace used for subsequent commands
kubectl config set-context --current --namespace=myns
```

### Lab 5.6 Namespaces

1. Create a namespace, see [create namespace docs](https://kubernetes.io/blog/2015/08/using-kubernetes-namespaces-to-manage/#creating-a-new-namespace)
2. Create a webserver Pod in the namespace
3. Review created resources to confirm namespace assigned to the Pod
4. Delete resources created

<details>
<summary>lab5.6 solution</summary>

```sh
kubectl create ns myns --dry-run=client -o yaml > lab5-6.yaml
echo --- >> lab5-6.yaml
kubectl run mypod --image=httpd -n myns --dry-run=client -o yaml >> lab5-6.yaml
kubectl apply -f lab5-6.yaml
kubectl get pods
kubectl describe -f lab5-6.yaml | less
kubectl delete -f lab5-6.yaml
```
</details>

## 6. Exploring Pods

Whilst a Pod is running, the _kubelet_ is able to restart containers to handle some faults. Within a Pod, Kubernetes tracks different container states and determines what action to take to make the Pod healthy again.

Kubernetes tracks the _phase_ of a Pod

- Pending - Pod starts here and waits to be scheduled, image download, etc
- Running - at least one container running
- Succeeded - all containers terminated successfully
- Failed - all containers have terminated, at least one terminated in failure
- Unknown - pod state cannot be obtained, either node communication breakdown or other

Kubernetes also tracks the _state_ of containers running in a Pod

- Waiting - startup not complete
- Running - executing without issues
- Terminated - ran into issues whilst executing

### Debugging Pods

The first step in debugging a Pod is taking a look at it. Check the current state of the Pod and recent events with:

```sh
kubectl describe pods $POD_NAME
```

When running commands locally in a Terminal, you can immediately see the output `STDOUT`. However, applications running in a cloud-native environment have their own way of showing their outputs - for Kubernetes, you can view a Pod `STDOUT` with:

```sh
kubectl logs $POD_NAME
```

> A Pod `STATUS=CrashLoopBackOff` means the Pod is in a cool off period following container failure. The container will be restarted after cool off \
> You will usually find more clues in the logs when a Pod shows a _none-zero_ `Exit Code` \
> See the [official _debug running pods_ tutorial](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/) for more details

### Lab 6.1 Troubleshoot failing Pod

1. Create a Pod with mysql image and confirm Pod state
2. Get detailed information on the Pod and review Events (any multiple attempts?), 'State', 'Last State' and their Exit codes.
   - Note that Pod `STATES` might continue to change for containers in error due to default `restartPolicy=Always`
3. Review cluster logs for the Pod
4. Apply relevant fixes until you have a mysql Pod in 'Running' state
5. Delete created resources

<details>
<summary>lab6.1 solution</summary>

```sh
kubectl run mydb --image=mysql --dry-run=client -o yaml > lab6-1.yaml
kubectl apply -f lab6-1.yaml
kubectl get pods
kubectl describe -f lab6-1.yaml | less
# use the `watch` program to rerun kubectl command every 2secs, see `watch --help`
watch minikube kubectl -- get pods # if not using alias `watch kubectl get pods`
ctrl+c
kubectl delete -f lab6-1.yaml
kubectl run mydb --image=mysql --env="MYSQL_ROOT_PASSWORD=secret" --dry-run=client -o yaml > lab6-1.yaml
kubectl apply -f lab6-1.yaml
kubectl get pods
kubectl describe -f lab6-1.yaml | less
kubectl delete -f lab6-1.yaml
```
</details>

### Ephemeral containers

[Ephemeral containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) are useful for interactive troubleshooting when kubectl exec is insufficient because a container has crashed or a container image doesn't include debugging utilities, such as with [distroless images](https://github.com/GoogleContainerTools/distroless).

```sh
# create a `mysql` Pod called `mypod` (assume the pod fails to start)
kubectl run mydb --image=mysql
# add ephemeral container to Pod `mypod`
kubectl debug -it ephemeral-pod --image=busybox:1.28 --target=ephemeral-demo
```

> The `EphemeralContainers` feature must be enabled in the cluster and the `--target` parameter must be supported by the _container runtime_ \
> When not supported, the _Ephemeral Container_ may not be started, or started without revealing processes

### Port forwarding

Port forwarding in Kubernetes should only be used for testing purposes.

```sh
# get a list of pods with extra information, including IP Address
kubectl get pods -o wide
# forward host computer's port 8080 to container `mypod` port 80, requires `ctrl+c` to terminate
kubectl port-forwarding mypod 8080:80
```

> When a program runs in a unix-based environment, it starts a process. A _foreground process_ prevents further execution of commands, e.g. `sleep`

```sh
# run any foreground command in the background by adding an ampersand &
sleep 60 &
# view running background processes and their ids
jobs
# bring a background process to the foreground
fg $ID
# run the `kubectl port-forwarding` command in the background
kubectl port-forwarding mypod 8080:80 &
```

### Lab 6.2. Expose container port

1. Create a webserver Pod
2. View created resources and determine Pod IP address
3. Access the webserver with the IP address (you can use `curl`)
4. Use port forwarding to access the webserver on http://localhost:5000
5. Terminate port forwarding and delete created resources

<details>
<summary>lab6.2 solution</summary>

```sh
kubectl run webserver --image=httpd
kubectl get pods -o wide
curl $POD_IP_ADDRESS
kubectl port-forward webserver 5000:80 &
curl localhost:5000
fg 1
ctrl+c
kubectl delete pods webserver
```
</details>

### Security context

> This section requires a basic understanding of unix-based systems file permissions and access control covered in [ch2 - container access control](#docker-container-access-control)

A [security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) defines privilege and access control settings for a Pod or Container. Security context can be controlled at Pod-level `pod.spec.securityContext` as well as at container-level `pod.spec.containers.securityContext`. A detailed explanation of security context is provided in the linked docs, however, for CKAD, we will only focus on the following:

- `runAsGroup: $GID` - specifies the GID of logged-in user in pod containers (pod and container level)
- `runAsNonRoot: $boolean` - specifies whether the containers run as a non-root user at image level - containers will not start if set to `true` while image uses root (pod and container)
- `runAsUser: $UID` - specifies the UID of logged-in user in pod containers (pod and container)
- `fsGroup: $GID` - specifies additional GID used for filesystem (mounted volumes) in pod containers (pod level)
- `privileged: $boolean` - controls whether containers will run as privileged or unprivileged (container level)
- `allowPrivilegeEscalation: $boolean` - controls whether a process can gain more privileges than its parent process - always `true` when the container is run as privileged, or has `CAP_SYS_ADMIN` (container level)
- `readOnlyRootFilesystem: $boolean` - controls whether the container has a read-only root filesystem (container level)

> 

```sh
# show pod-level security context options
kubectl explain pod.spec.securityContext | less
# show container-level security context options
kubectl explain pod.spec.containers.securityContext | less
# view pod details for `mypod`
kubectl get pods mypod -o yaml
```

### Lab 6.3. Set Pod security context

Using the official docs manifest example `pods/security/security-context.yaml` as base to:

1. Explore and compare the security context options available at pod-level vs container-level
2. Use the official manifest example `pods/security/security-context.yaml` as base to create a Pod manifest with these security context options:
   - all containers have a logged-in user of `UID: 1010, GID: 1020`
   - all containers set to run as non-root user
   - mounted volumes for all containers in the pod have group `GID: 1110`
   - escalating to root privileges is disabled ([more on privilege escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/))
3. Apply the manifest file and review details of created pod
4. Review pod details and confirm security context applied at pod-level and container-level
5. Connect an interactive shell to a container in the pod and confirm the following:
   - current user
   - group membership of current user
   - ownership of entrypoint process
   - ownership of the mounted volume `/data/demo`
   - create a new file `/data/demo/new-file` and confirm file ownership
   - escalate to a shell with root privileges `sudo su`
6. Edit the pod manifest file to the following:
   - do not set logged-in user UID/GID
   - do not set root privilege escalation 
   - all containers set to run as non-root user
7. Create a new pod with updated manifest
8. Review pod details and confirm events and behaviour
   - what were your findings?
9. Delete created resources

<details>
<summary>lab6.3 solution</summary>

```sh
# host terminal
kubectl explain pod.spec.securityContext | less
kubectl explain pod.spec.containers.securityContext | less
wget -qO lab6-3.yaml https://k8s.io/examples/pods/security/security-context.yaml
nano lab6-3.yaml # edit file as below
#spec:
#  securityContext:
#    runAsUser: 1010
#    runAsGroup: 1020
#    fsGroup: 1110
#  containers:
#  - name: sec-ctx-demo
#    securityContext:
#      allowPrivilegeEscalation: false
kubectl apply -f lab6-3.yaml
kubectl describe pods security-context-demo | less
kubectl get pods security-context-demo -o yaml | grep -A 4 -E "spec:|securityContext:" | less
kubectl exec -it security-context-demo -- sh
# container terminal
whoami
id # uid=1010 gid=1020 groups=1110
ps
ls -l /data # root 1110
touch /data/demo/new-file
ls -l /data/demo # 1010 1110
sudo su # sudo not found - an attacker might try other ways to gain root privileges
exit
# host terminal
nano lab6-3.yaml
#spec:
#  securityContext:
#    runAsNonRoot: true
#    fsGroup: 1110
#  containers:
#  - name: sec-ctx-demo
#    securityContext:
#      allowPrivilegeEscalation: false
kubectl delete -f lab6-3.yaml
kubectl apply -f lab6-3.yaml
kubectl get pods security-context-demo
kubectl describe pods security-context-demo | less
# found error creating container - avoid conflicting rules, enforcing non-root user `runAsNonRoot: true` requires a non-root user specified `runAsUser: $UID`
```
</details>

### Jobs

Pods always restarts their containers on failure and maintain a 'Running' status. A Job is a way to run a Pod until certain tasks are complete, a 'Completed' status, without restarting their containers, e.g. backup. Jobs creates Pods for the task. \
Job types are determined by the `completions` and `parallelism` values:

```sh
# one pod started per job, unless failure
completions=1; parallelism=1
# multiple pods started, until one successfully completes task
completions=1; parallelism=x
# multiple pods started, until n successful task completions
completions=n; parallelism=x
# clean up jobs automatically
spec.ttlSecondsAfterFinished
# explore jobs doc
kubectl create job -h
# create a job
kubectl create job datejob --image=busybox -- date
```

### CronJobs

CronJobs are used to run recurring tasks for `n` number of times. CronJobs creates Jobs to run the tasks.

```sh
# cronjob time syntax: * * * * * - minute hour day_of_month month day_of_week
kubectl create cronjob -h
# create a cronjob
kubectl create cronjob cj --image=busybox --schedule="* * * * *" -- date
# view details on autocleanup of jobs created by cronjobs
kubectl explain cronjobs.spec.jobTemplate.spec
```

### Lab 6.4. Working with jobs

- Create a job that uses a busybox image to run the `date` command. Review output of the job.
- Create a recurring job that uses busybox to run the `date` command every 10s. Cleanup jobs after 1min of completion. Review output of jobs.

### Limits and Requests

Request is the initial/minimum value of resources provided to a Pod, while Limit is the maximum value of resources available to a Pod. See [container resource management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for more details.

> A Pod remains in "Pending" status until a Node with sufficient resources becomes available

```sh
# limit types
spec.containers[].resources.limits.cpu # in cores and millicores, 500m = 0.5 CPU
spec.containers[].resources.limits.memory # Ki (1024) | Mi | Gi | Ti | Pi | Ei | m | "" | k (1000) | M | G | T | P | E
spec.containers[].resources.limits.hugepages-<size>
# request types
spec.containers[].resources.requests.cpu
spec.containers[].resources.requests.memory
spec.containers[].resources.requests.hugepages-<size>
```

### Lab 6.5. Resource limitation

- Create a Pod manifest file Pod with the following:
  - runs in `dev` namespace
  - assign environment variable `NODE_ENV=development`
  - uses `busybox` image that runs command `printenv` with arguments `NODE_ENV`
  - restart only on failure
  - initial - 0.25 CPU, 64 mebibytes
  - maximum - 0.5 CPU, 128 mebibytes
- View created pod details, status and output

## 7. Deployments

[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) manages Pods with scalability and reliability, this is the standard way to manage Pods in live environments.

```sh
# create a deployment with specific replicas (default replicas=1), see `kubectl create deploy --help`
kubectl create deploy myapp --image=nginx --replicas=3
# show details of deployment, see `kubectl describe deploy --help`
kubectl describe deploy [deploymentName]
# scale deployment
kubectl scale deployment myapp --replicas=4
# edit deployment (not all fields are edittable), see `kubectl edit deployment -h`
kubectl edit deployment myapp -o yaml --save-config
```

### Lab 7.1. Deploy an app

- Create a simple deployment
- Show more details of the deployment and review the following:
  - namespace, labels, selector, replicas, update strategy type, pod template, conditions, {old|new}replicaset and events
- Show all the resources created by the deployment - filter by the default selector
- Delete a pod and confirm behaviour by reviewing resources left, pods and their statuses
- Delete a replicaset and confirm behaviour by reviewing resources left, pods, replicasets, etc, and their properties

### Labels, selectors and annotations

Labels are used for groupings, filtering and providing metadata. Selectors are used to group related resources. Annotations are used to provide additional metadata but are not used in queries.

When a deployment is created directly, a default label `app=[appName]` is assigned. When a pod is created directly, a default label `run=[podName]` is assigned

> Labels added after creating a deployment are not inherited by the resources 

```sh
# add label
kubectl label deployment myapp state=test
# show list of created deployments and their labels
kubectl get deployments --show-labels
# show list of all created resources and their labels
kubectl get all --show-labels
# show deployments filtered by specific label
kubectl get deployments --selector state=test
# show all resources filtered by specific label
kubectl get all --selector app=myapp
# remove a label by a minus sign after label key
kubectl label pod [podName] app-
```

### Lab 7.2. Working with labels

- Create a deployment
- Add a new label to the deployment
- Show more details of the deployment and review how labels are assigned
- Show a list of deployments and their labels
- Show a list of all resources and their labels, filtered by default label
- Show a list of all resources and their labels, filtered by new label added, compare with above
- Remove the default label from pods in the deployment and review behaviour
  - Show a list of pods and their labels

### Scaling deployments

A deployment creates a ReplicaSet that manages scalability.

> Do not manage replicasets outside of deployments

```sh
# view api-versions
kubectl api-versions
# view api-versions and their resource types
kubectl api-resources
# scale deployment manually by specifying number of replicas
kubectl scale deployment [deploymentName] --replicas=n
# scale deployment manually by editing YAML file - note not all fields are edittable
kubectl edit deploy [deploymentName]
```

### Lab 7.3. Scale an app

- Create a deployment with 3 replicasets
  - how do you verify the number of replicasets?
  - show all created resources filtered by default selector
- Copy a deployment template from [official docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
  - what happens when you create a deployment from manifest file with an incorrect version? Use template with `apiVersion: apps/v0.5`
  - what happens when you create a deployment from manifest file with an old version? Use template with `apiVersion: v1`
  - how do you confirm you have the correct versions?
  - create a deployment with the template
- Scale the deployment manually by editing the YAML file
  - change an edittable field and save
  - change a non-edittable field, save and review behaviour

### Update strategy

[Rolling updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) is the default deployment strategy and can be used to update any of the fields available to the `kubectl set` command. A new ReplicaSet is created for the update, and the old ReplicaSet is scaled to 0 after successful update. By default, 10 old ReplicaSets will be kept, see [`deployment.spec.revisionHistoryLimit`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#revision-history-limit)

The other type of update strategy is [Recreate](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#recreate-deployment), where all Pods are killed before nre Pods are created. This is useful when you cannot have different versions of an app running simultaneously, e.g database.

Rolling update option [`maxUnavailable`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable) is used to control number of Pods upgraded simultaneously, while [`maxSurge`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge) controls the number of Pods in addition to the replicas specified during update. Aim to have a higher `maxSurge` than `maxUnavailable`. These options can be set directly in YAML file `spec.strategy.rollingUpdate`

> Scaling down a deployment to 0 is another way to delete all resources associated while keeping the deployment and ReplicaSet config for a quick scale up when required.

```sh
# edit some fields by setting directly, see `kubectl set -h`
kubectl set image deploy myapp nginx=nginx:1.24
kubectl set env deploy myapp dept=MAN
# show recent update history
kubectl rollout history deployment myapp -h
# show update events
kubectl describe deployment myapp
# revert an update
kubectl rollout undo -h
# view rolling update options
kubectl get deploy myapp -o yaml
# view all deployments history, see `kubectl rollout -h`
kubectl rollout history deployment
# view specific deployment history
kubectl rollout history deployment [deploymentName]
# view specific change revision/log for the deployment
kubectl rollout history deployment [deploymentName] --revision=n
# revert deployment to previous version/revision
kubectl rollout undo deployment [deploymentName] --to-revision=n
```

### Lab 7.4. Rolling updates

- Create a YAML file for `nginx:1.23` deployment and 3 replicas, with label `lab=updates`, max 2 Pods can be updated simultaneously
- Create and review details of the deployment
  - by default, how many pods can be upgraded simultaneously during update?
  - by default, how many pods can be created in addition to the number of replicas during update?
  - show a list of deployments and their labels
  - show only resouces specific to the deployment
  - view deployment history
- Downgrade the image by directly setting a new image version, use `kubectl set`
- Review details of the deployment
  - review behaviour by showing only resources specific to the deployment
  - confirm changes applied
  - view deployment history specific to this deployment
  - note that `change-cause` is populated by the record-option which isn't used
- View first and second change revisions for the deployment
- View deployment history specific to this deployment
- Show details of the deployment in YAML format
  - review `spec.strategy.type` and `spec.strategy.rollingupdate` fields
- Scale the deployment to 8 replicas and review the following:
  - pods READY and STATUS fields
  - deployment READY UP-TO-DATE and AVAILABLE fields
  - replicaset DESIRED CURRENT READY and AGe fields
  - view deployment history specific to this deployment
- Add an environment variable to the deployment
  - show all resources filtered by label=app and review resource states as above
  - view deployment history specific to this deployment
- Revert deployment to the second revision, see `kubectl rollout -h`
  - view deployment history specific to this deployment
- Scale the replicasets to 0 and review behaviour
  - why would you scale to 0 instead of delete?
  - is the scale down captured by rollout history?

### Deployment Types

There are two kinds of deployments, DaemonSet and StatefulSet.

A DaemonSet ensures that all (or some) Nodes run a copy of a particular Pod. This is useful in a multi-node cluster where specific application is required on all nodes, e.g. running a  - cluster storage, logs collection, node monitoring, network agent - daemon on every node. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created. DaemonSets can only be created by YAML file.

```sh
# create daemonset via yaml file
kubectl create -f daemonset.yaml
# view daemonsets pods
kubectl get ds,pods
# view daemonset in kube system namespace
kubectl get ds,pods -n kube-system
```

### Lab 7.5. DaemonSet

Create a DaemonSet by using a [template from official docs](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- Review the `Kind` in the template before creation
- Review the DaemonSet pods created
- Review the DaemonSet in kube-system

### Lab 7.6. Autoscaling

Autoscaling is used in real clusters but not covered in CKAD. See [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server) for more details and see [HorizontalPodAutoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#run-and-expose-php-apache-server) for complete lab.

```sh
# install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
# docker-desktop disable certificate validation using `--kubelet-insecure-tls`
kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
# delete metrics server
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## 8. Networking

### Service

A service provides access to applications by exposing them. Uses round-robin load balancing. Service targets Pods by selector. Services exists independent from deployment, is not deleted during deployment deletion and can provide access to Pods in multiple deployments.

Kube-proxy agent listens for new Services and Endpoints on a random port and redirects traffic to Pods specified as Endpoints

> Understand the Kubernetes network architecture: Pod Net - Cluster Net - Node Net - Ingress
> Understand the Kubernetes microservices architecture: Database - Backend - Frontend

#### Service Types

- ClusterIP service type is the default, exposes the service on an internal Cluster IP address
- NodePort service type provides public access to the application through host port forwarding
- LoadBalancer for cloud service (not for CKAD)
- ExternalName for DNS names (not for CKAD)

```sh
# view default services/pods
kubectl get svc,pods -n kube-system
# create a service by exposing a type, see `kubectl expose -h`
kubectl expose [type=deploy,svc,rc,rs,etc] [typeName] --port=PORT
# create a service by expose a deployment on port 80
kubectl expose deploy [deploymentName] --port=80
# view more details of the service to confirm properties
kubectl describe svc [name]
# view details of service in yaml
kubectl get svc [name] -o yaml | less
				   
			   
# edit service
kubectl edit svc [name]
```

### Lab 8. Exposing your application internally

Use Minikube locally

- Create a single replica nginx deployment called `nginx`
- Scale the deployment to 3 replicas
- Confirm all resources created
- Create a service for the deployment on port 80
  - Confirm all resources created, note service TYPE, CLUSTER-IP and PORT(S)
  - View more details of the service, note IPs, Port, TargetPort and Endpoints
  - Compare with the YAML output of the service
  - View the endpoints created
  - Access the app from the container host via `minikube ssh`
  - Access the app from a browser via `port-forwarding`
- Change the service type to NodePort of `nodePort=32000`
  - Can you access the app through the NodePort `$(minikube ip):32000`?
- Create a naked Pod, e.g. `busybox`
  - list the pods created
  - run command inside the Pod `cat /etc/resolv.conf` to compare DNS server IP
  - run command inside the Pod `nslookup [deploymentName]` to compare Gateway/Domain server IP
  - what IP addresses do these match?

## 9. Ingress

### Ingress rules

This is similar to defining API routes on a backend application, except that each defined route points to a deployment.

> `kubectl create ingress myingress --rule="/=app1:80" --rule="/about=app2:3000" --rule="/contact=app3:8080"`

- if no host is specified, the rule applies to all inbound HTTP traffic
- routes/paths can be defined with a POSIX regex
- `pathType` can be `Exact (default)` or `Prefix` based matching
- each path points to a service or a resource.
- a default path can be defined for traffic that doesn't match any specified paths, similar to a 404 route

### Ingress Types

- **single-service ingress** defines a single rule to access a single service
- **simple fanout ingress** defines two or more rules of different routes/paths to access different services
- **name-based virtual hosting ingress** defines two or more rules with dynamic routes based on host header, requires a DNS entry for each host header

> name-based example: `kubectl create ingress namebased --rule="app.domain.com/*=appservice:80" --rule="api.domain.com/*=apiservice:80" --rule="db.domain.com/*=dbservice:80"`

### Basic commands

```sh
# enable ingress manually
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/cloud/deploy.yaml
# list existing minikube addons
minikube addons list
# enable ingress on minikube
minikube addons enable ingress
# confirm ingress namespace added
kubectl get ns
# confirm resources in ingress namespace
kubectl get all -n ingress-nginx
# create ingress, see `kubectl create ingress -h`
kubectl create ingress [name] --rule="[path]=[deploymentName]:port"
kubectl create ingress nginx-ingress --rule="/web=nginx-app:80"
```

### Lab 9.1. Managing Ingress

- Enable Ingress
  - confirm new namespace added
  - confirm all resources in ingress namespace
- Create a new `nginx` deployment and expose as NodePort port 80, use YAML file
  - verify access on `$(minikube ip)`
- Add `[deploymentName.info` to `/etc/hosts` mapped to `minikube ip`
- Create an ingress for the `nginx` deployment on path `/`, use YAML file
  - verify access
- Create two new deployments of any web server images called `cat` and `dog`, use YAML file
  - expose both deployments as `Cluster-IP` on port 80
  - add entries in `/etc/hosts` for `cat.domain.com` and `dog.domain.com` both mapped to `$(minikube ip)`
- Edit existing Ingress, use YAML file
  - add rules for both deployments using their subdomains on path `/cat` and `/dog` respectively, and `pathType=Prefix` for both
- Verify access to the new deployments via subdomains	

### Network policies

In Kubernetes, Pods can always communicate through namespaces. All traffics are allowed until you have a [NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

Network policies are implemented by the [network plugin](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/). This requires a networking solution which supports NetworkPolicy, otherwise, will have no effect. Network policies are additive.

Minikube needs to be started with the `--cni=calico` flag to use the Calico network plugin.

> The Calico plugin conflicts with Ingress and thus should be disabled after this lab.

```sh
minikube stop
minikube delete
# start minikube with calico plugin
minikube start --kubernetes-version=1.23.9 --cni=calico
# verify calico plugin running
kubectl get pods -n kube-system
# allow plenty of time (+5mins) for kubedns and calico to enter `running` status
```

> Allow plenty of time (+5mins) for kubedns and calico to enter `Running` status

### Network policy identifiers

There are three different identifiers that controls entities that a Pod can communicate with:

- `podSelector`: Pod allows access to other Pods with the matching selector label (note: a Pod cannot block itself)
- `namespaceSelector`: Pod allows incoming traffic from namespaces with the matching selector label
- `ipBlock`: specifies a range of cluster-external IPs to allow access (not for CKAD - note: node traffic is always allowed)

```sh
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
# create default deny all ingress/egress traffic
spec:
  podSelector: {}
  policyTypes:
  - Ingress # or Egress
# allow all ingress/egress traffic
spec:
  podSelector: {}
  ingress: # or egress
  - {}
  policyTypes:
  - Ingress # or Egress
```

### Lab 9.2. Declare network policy

Follow the [official declare network policy walkthrough](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

> Note: prepend `https://k8s.io/examples/` to any example files in the official docs to use the file with `kubectl`	

## 10. Storage

[PersistentVolume (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes, with a lifecycle independent of any individual Pod that uses the PV.

PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Claims can request specific size and [access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany, or ReadWriteOncePod)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes).

- Pods connect to the PVC, and a PVC connects to the PV, both in a 1-1 relationship (only one PVC can connect to a PV)
- PVC can be created from an existing PVC
- PVC will remain in `STATUS=Pending` until it finds and connects to a matching PV and thus **`STATUS=Bound`** 
- PV supports a number of [raw block volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)

### Access modes

- ReadWriteOnce: volume can be mounted as read-write by a single node - allows multiple pods running on the node access the volume
- ReadOnlyMany: volume can be mounted as read-only by many nodes
- ReadWriteMany: volume can be mounted as read-write by many nodes
- ReadWriteOncePod: volume can be mounted as read-write by a single Pod

### PV and PVC attributes

| PV attributes    | PVC attributes   |
|------------------|------------------|
| capacity         | resources        |
| volume modes     | volume modes     |
| access modes     | access modes     |
| storageClassName | storageClassName |
| mount options    | selector         |
| reclaim policy   |                  |
| node affinity    |                  |
| phase            |                  |

### Storage Class

A [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) provides a way for administrators to describe the "classes" of storage they offer. It enables automatic PV provisioning to meet PVC requests, thus removing the need to manually create PVs. StorageClass must have a specified **provisioner** that determines what volume plugin is used for provisioning PVs.

- A PV with a specified `storageClassName` can only be bound to PVCs that request that `storageClassName`
- A PV with `storageClassName` attribute not set is intepreted as _a PV with no class_, and can only be bound to PVCs that request a PV with no class.
- A PVC with `storageClassName=""` (empty string) is intepreted as _a PVC requesting a PV with no class_.
- A PVC with `storageClassName` attribute not set is not quite the same and behaves different whether [the `DefaultStorageClass` admission plugin is enabled](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
  - if the admission plugin is enabled, and a default StorageClass specified, all PVCs with no `storageClassName` can be bound to PVs of that default
  - if a default StorageClass is not specified, PVC creation is treated as if the admission plugin is disabled
  - if the admission plugin is disabled, all PVCs that have no `storageClassName` can only be bound to PVs with no class

> If a PVC doesn't find a PV with matching access modes and storage, StorageClass may dynamically create a matching PV

### Basic commands

```sh
# list PVCs, PVs
kubectl get {pvc|pv|storageclass}
# view more details of a PVC
kubectl decribe {pvc|pv|storageclass} [name]
# hostPath PersistentVolume
https://k8s.io/examples/pods/storage/pv-volume.yaml
# hostPath PersistentVolumeClaim
https://k8s.io/examples/pods/storage/pv-claim.yaml
```

> `hostPath` volumes is created on the host, in minikube use the `minikube ssh` command to access the host (requires starting the cluster with `--driver=docker`)

### Lab 10.1. PVs and PVCs

Use the `hostPath` PV and PVC above as base.

- Create a PV from a YAML file with 3Gi capacity
- Create a PVC from a YAML file requesting 1Gi capacity
  - What `STATUS` does the PVC have?
  - Does the PVC use the existing PV and why or why not?
- What happens when a PVC is created without specifying a `StorageClass` in the YAML file?

### Lab 10.2. Configuring Pods storage with PVs and PVCs

The benefit of configuring Pods with PVCs is to decouple site-specific details.

You can follow the [official "configure a Pod to use a PersistentVolume for storage" docs walkthrough](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume) to complete this lab.

1. Create a `/mnt/data/index.html` file on cluster host `minikube ssh` with some message, e.g. "Hello, World!"
2. Create a PV with the following parameters, see `https://k8s.io/examples/pods/storage/pv-volume.yaml`
   - uses `hostPath` storage
   - allows multiple pods access the storage
3. Create a Pod running a webserver to consume the storage, see `https://k8s.io/examples/pods/storage/pv-pod.yaml`
  - uses PVC, see `https://k8s.io/examples/pods/storage/pv-claim.yaml`
  - image is `httpd` and default documentroot is `/usr/local/apache2/htdocs` or `/var/www/html`
4. Verify all resources created `pod,pv,pvc,storageclass`, and also review each detailed information
   - review `STATUS` for PV and PVC
   - did the PVC in [3] bind to the PV in [2], why or why not?
5. Connect to the Pod via an interactive shell and confirm you can view the contents of cluster host file `curl localhost`
6. Clean up all resources created

<details>
  <summary><b>spoiler: lab steps</b></summary>

```sh
minikube ssh
sudo mkdir /mnt/data
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
cat /mnt/data/index.html
exit # exit ssh
echo --- > lab10-2.yaml
wget https://k8s.io/examples/pods/storage/pv-volume.yaml -O- >> lab10-2.yaml
echo --- >> lab10-2.yaml
wget https://k8s.io/examples/pods/storage/pv-claim.yaml -O- >> lab10-2.yaml
echo --- >> lab10-2.yaml
wget https://k8s.io/examples/pods/storage/pv-pod.yaml -O- >> lab10-2.yaml
echo --- >> lab10-2.yaml
nano lab10-2.yaml # edit the final file accordingly
kubectl apply -f lab10-2.yaml
kubectl get pod,pv,pvc,storageclass
kubectl describe pod,pv,pvc,storageclass | less
kubectl exec -it task-pv-pod -- /bin/bash
kubectl delete -f lab10-2.yaml
```
</details>

For further learning, see [mounting the same persistentVolume in two places](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#mounting-the-same-persistentvolume-in-two-places) and [access control](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#access-control)

## 11. ConfigMaps and Secrets

### Variables

Variables can be specified via the command-line when creating a _naked_ Pod with `kubectl run mypod --image=nginx --env="MY_VARIABLE=myvalue"`. However _naked_ Pods are not recommended in live environments, so our main focus is creating variables for deployments.

The `kubectl create deploy` command does not currently support the `--env` option, thus the easiest way to add variables to a deployment is to use `kubectl set env deploy` command after the deployment is created.

> Note that doing `kubectl set env deploy --dry-run=client` will only work if the deployment is already created \
> To generate a YAML file with variables via command-line, first `kubectl create deploy`, then `kubectl set env deploy --dry-run=client -o yaml` and edit to remove unnecessary metadata and statuses

### Lab 11.1. Deployment variables

1. Create a databse deployment using `MYSQL` image
2. Troubleshoot and fix any deployment issues to get a running `STATUS`

### ConfigMaps

ConfigMaps are used to decouple configuration data from application code. The configuration data may be variables, files or command-line args.

- ConfigMaps should be created before creating an application that relies on it
- A ConfigMap created from a directory includes all the files in that directory and the default behaviour is to use the filenames as keys

```sh
# create configmap from file or directory (`--from-file` can be used multiple times), see `kubectl create cm -h`
kubectl create {cm|configmap} [cmName] --from-file=path/to/file/or/directory
# create configmap from file with specified key
kubectl create {cm|configmap} [cmName] --from-file=key=path/to/file
# create configmap from env-file
kubectl create {cm|configmap} [cmName] --from-env-file=path/to/file.env
# create configmap from literal values
kubectl create cm --from-literal=KEY1=value1 --from-literal=KEY2=value2
# display configmap details
kubectl get cm [cmName] -o yaml
kubectl describe cm [cmName]
# use configmap in deployment (can be used with `--dry-run=client`), see `kubectl set env -h`
kubectl set env deploy [deploymentName] --from=configmap/[cmName]
```

### Lab 11.2. ConfigMaps as variables

1. Create a `file.env` file with the variables needed for `MYSQL` from [Lab 11.1](#lab-111-deployment-variables)
2. Create a ConfigMap from the file
3. Create a `MYSQL` deployment using the ConfigMap and troubleshoot Pod status
4. Review the deployment as YAML and note how the variables are created

### Lab 11.3. Mounting ConfigMaps

You may also follow the [offical "add ConfigMap data to a Volume" docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)

1. Create a `index.html` file with some content
2. Create a ConfigMap from the file and verify resources created
3. Create a webserver deployment and mount the file to the DocumentRoot via ConfigMap
   - use `https://k8s.io/examples/pods/pod-configmap-volume.yaml` as base
   - option `nginx` DocumentRoot - /usr/share/nginx/html
   - option `httpd` DocumentRoot - /usr/local/apache2/htdocs
4. Connect a shell to the container and confirm your file is being served

### Secrets

Secrets are similar to ConfigMaps but specifically intended to hold sensitive data such as passwords, auth tokens, etc. By default, k8s secrets are unencrypted but base64 encoded.

To safely use secrets, ensure to:

1. [Enable Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets.
2. [Enable or configure RBAC rules](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) to
   - restrict read/write
   - limit access to create/replace secrets
   
Uses of secrets

- [as files](https://kubernetes.io/docs/concepts/configuration/secret/?ref=faun#using-secrets-as-files-from-a-pod) which may be mounted in Pods, e.g. accessing secret data in a Pod, TLS, etc
  - consider using `defaultMode` when mounting secrets to set file permissions to `user:readonly - 0400`
  - mounted secrets are automatically updated in the Pod when they change
- [as container environment variable](https://kubernetes.io/docs/concepts/configuration/secret/?ref=faun#using-secrets-as-environment-variables) which may be managed with `kubectl set env`
- [as `image registry credentials`](https://kubernetes.io/docs/concepts/configuration/secret/?ref=faun#using-imagepullsecrets), e.g. docker image registry creds

> Secrets are basically encoded [ConfigMaps](#configmaps) and are both managed with `kubectl` in a similar way, see `kubectl create secret -h` for more details

```sh
# secret as file for tls keys, see `kubectl create secret tls -h`
kubectl create secret tls [secretName] --cert=path/to/file.crt --key=path/to/file.key
# secret as file for ssh private key, , see `kubectl create secret generic -h`
kubectl create secret generic [secretName] --from-file=ssh-private-key=path/to/id_rsa
# secret as env-var for passwords
kubectl create secret generic [secretName] --from-literal=[PASSWORD_KEY]=passwordvalue
# secrets as image registry creds (the `docker-registry` option works for all registry types)
kubectl create secret docker-registry [secretName] --docker-username=[username] --docker-password=[password] --docker-email=[email] --docker-server=[image-registry-url]
# view details of the secret
kubectl get secret [secretName] -o yaml
# view the contents of the secret
kubectl get secret [secretName] -o jsonpath='{.data}'
# view the contents of a specified key in the secret, e.g.  '{"game":{".config":"yI6eyJkb2NrZXIua"}}'
kubectl get secret [secretName] -o jsonpath='{.data.game.\.config}'
# decode the `base64` encoded secret
kubectl get secret [secretName] -o jsonpath='{.data.game.\.config}' | base --decode
echo [base64string] | base64 --decode
```

> See the [Kubernetes JSONPath support docs](https://kubernetes.io/docs/reference/kubectl/jsonpath/) to learn more about using `jsonpath`

### Lab 11.4. Decoding secrets

You may follow the [official "managing secrets using kubectl" docs](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)

1. Review the `coredns` Pod in `kube-system` namespace in YAML and determine the name of the `ServiceAccount`
2. Review the `ServiceAccount` in YAML and determine the name of the `Secret`
3. View the contents of the `Secret` and decode the value of its `namespace` key.

### Lab 11.5. Secrets as environment variables

Repeat [lab 11.2](#lab112-configmaps-as-variables) with secrets

> See the [official "secrets as container env-vars" docs](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-as-container-environment-variables)

### Lab 11.6. Secrets as files

Repeat [lab 11.3](#lab113-mounting-configmaps) with secrets.

> See the [official "using secrets as files" docs](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)

### Lab 11.7. Secrets as docker registry credentials

1. Create a secret with the details of your docker credentials
2. View more details of the resource created `kubectl describe`
3. View details of the secret in `yaml`
4. Decode the contents of the `.dockerconfigjson` key with `jsonpath`

> See the [official "create an `imagePullSecret`" docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#create-an-imagepullsecret)

## 12. K8s API

### Understanding the API

Use `kubectl api-resources | less` for an overview of the API.

- `APIVERSION`
  - `v1` core kubernetes API group
  - `apps/v1` first extension to the core group
  - during deprecation/transition, multiple versions of the same resource may be available, e.g. `policy/v1` and `policy/v1beta1`
- `NAMESPACED` controls visibility

> The Kubernetes **release cycle is 3 months** and deprecated features are supported for a minimum of 2 release cycles (6 months).
> Respond to deprecation message swiftly, you may use **`kubectl api-versions`** to view a short list of API versions and **`kubectl explain --recursive`** to get more details on affected resources. \
> The current API docs at time of writing is `https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/`

#### kube-apiserver

The [Kubernetes API server `kube-apiserver`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) is the interface to access all Kubernetes features, which include pods, services, replicationcontrollers, and others. It provides the frontend to the cluster's shared state through which all other components interact. All **requests are secured by TLS certificates in `~/.kube/config`**.

From within a Pod, the API server is accessible via a Service named `kubernetes` in the `default` namespace. Therefore, Pods can use the **`kubernetes.default.svc`** hostname to query the API server.

The Kubernetes API server may authorize a request using one of [several authorization modes: Node, ABAC, RBAC or Webhook](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules).

#### kube-proxy

Working with direct access to a cluster node, like our minikube lab environment, removes the need for `kube-proxy`. When using the Kubernetes CLI `kubectl`, it uses stored TLS certificates in `~/.kube/config` to make secured requests to the `kube-apiserver`. However, direct access is not always possible with K8s in the cloud. The [Kubernetes network proxy `kube-proxy`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) runs on each node and make it possible to access `kube-apiserver` securely by other applications like `curl` or programmatically.

> See the [official "so many proxies" docs](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#so-many-proxies) for the different proxies you may encounter when using Kubernetes.

```sh
# view a more verbose pod detais
kubectl --v=10 get pods
# start kube-proxy
kubectl proxy --port=PORT
# explore the k8s API with curl
curl localhost:PORT/api
# get k8s version with curl
curl localhost:PORT/version
# list pods with curl
curl localhost:PORT/api/v1/namespaces/default/pods
# get specific pod with curl
curl localhost:PORT/api/v1/namespaces/default/pods/[podName]
# delete specific pod with curl
curl -XDELETE localhost:PORT/api/v1/namespaces/default/pods/[podName]
```

### Directly accessing the REST API

Two things are required to access a cluster - the **location** of the cluster and the **credentials** to access it. Thus far, we have used `kubectl` to access the API by running `kubectl` commands. The location and credentials that `kubectl` uses were automatically configured by Minikube during our [lab environment setup](#4-kubernetes-lab-environment).

Run `kubectl config view` to see the location and credentials configured for `kubectl`.

<details>
  <summary><code><b>kubectl config view</b></code></summary>
  
  ![image](https://user-images.githubusercontent.com/17856665/184899506-47878132-dc9b-4a6a-8491-080fe56c6261.png)
</details>

### Lab 12.1. Using kubectl proxy to access the API

Rather than run `kubectl` commands directly, we can use `kubectl` as a reverse proxy to provide the location and authenticate requests. See [access the API using kubectl proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#using-kubectl-proxy) for more details.

You may follow the [official "accessing the rest api" docs](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#directly-accessing-the-rest-api)

1. Expose the API with `kube-proxy`
2. Confirm k8s version information with `curl`
3. Explore the k8s API with curl
4. Create a deployment
5. List the pods created with `curl`
6. Get details of a specific pod with `curl`
7. Delete the pod with `curl`
8. Confirm pod deletion

### Checking API access

`kubectl` provides the `auth can-i` subcommand for [quickly querying the API authorization layer](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access).

```sh
# check if deployments can be created in a namespace
kubectl auth can-i create deployments --namespace dev
# check if pods can be listed
kubectl auth can-i get pods
# check if a specific user can list secrets
kubectl auth can-i list secrets --namespace dev --as dave
```

### Service Accounts

Just as user accounts identifies humans, a service account identifies processes running in a Pod.

- Service Accounts are the recommended way to authenticate to the API server within the k8s cluster
- A Pod created without specifying a ServiceAccount is automatically assigned the `default` ServiceAccount
- When a new ServiceAccount is created, a Secret is auto-created to hold the credentials required to access the API server
- The ServiceAccount credentials of a Pod are automounted with the Secret in each container within the Pod at:
  - token `/var/run/secrets/kubernetes.io/serviceaccount/token`
  - certificate (if available) `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`
  - default namespace `/var/run/secrets/kubernetes.io/serviceaccount/namespace`
- You can opt out of automounting API credentials for a ServiceAccount by setting `automountServiceAccountToken: false` on the ServiceAccount. Note that the pod spec takes precedence over the service account if both specify a `automountServiceAccountToken` value

### Lab 12.2. Accessing the API without kubectl proxy

This requires using the token of the default ServiceAccount. The token can be read directly (see [lab 11.4 - decoding secrets](#lab-114-decoding-secrets)), but the recommended way to get the token is via the [TokenRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/).

You may follow the [official "access the API **without** kubectl proxy" docs](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy).

1. Request the ServiceAccount token by YAML. You can also request by [`kubectl create token $SERVICE_ACCOUNT_NAME`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-token-em-) on Kubernetes v1.24+.
2. Wait for the token controller to populate the Secret with a token
3. Use `curl` to access the API with the generated token as credentials

<details>
  <summary><b>lab 12.2 steps</b></summary>

```sh
# request token
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: default-token
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
EOF
# confirm token generated (optional)
kubectl get secret default-token -o yaml
# use token
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
TOKEN=$(kubectl get secret default-token -o jsonpath='{.data.token}' | base64 --decode)
curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```
</details>

> Using `curl` with the `--insecure` option [skips TLS certificate validation](https://curl.se/docs/sslcerts.html)

### Lab 12.3. Accessing the API from a Pod without kubectl

You may follow the [official "access the API from within a Pod" docs](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/#without-using-a-proxy).

> From within a Pod, the Kubernetes API is accessible via the `kubernetes.default.svc` hostname

1. Connect an interactive shell to a container in a running Pod (create one or use existing)
2. Use `curl` to access the API at `kubernetes.default.svc/api` with the automounted ServiceAccount credentials (`token` and `certificate`)
3. Can you access the Pods list at `kubernetes.default.svc/api/v1/namespaces/default/pods`?

<details>
  <summary><b>lab 12.3 steps</b></summary>

```sh
# connect an interactive shell to a container within the Pod
kubectl exec -it $POD_NAME -- /bin/sh
# use token stored within container to access API
SA=/var/run/secrets/kubernetes.io/serviceaccount
CERT_FILE=$($SA/ca.crt)
TOKEN=$(cat $SA/token)
curl --cacert $CERT_FILE --header "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api
```
</details>

### RBAC

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization. RBAC authorization uses the `rbac.authorization.k8s.io` [API group](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning) for dynamic policy configuration through the Kubernetes API. RBAC is beyond CKAD, however, a basic understanding of RBAC can help understand ServiceAccount permissions.

The RBAC API declares four kinds of Kubernetes object: [Role, ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole), [RoleBinding and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding).

- Role is a namespaced resource; when you create a Role, you have to specify its namespace
- ClusterRole, by contrast, is a non-namespaced resource
- RoleBinding grants the permissions defined in a Role/ClusterRole to a user or set of users within a specific namespace
- ClusterRoleBinding grants permissions that access cluster-wide.

### ServiceAccount permissions

The Default RBAC policies grant scoped permissions to control-plane components, nodes, and controllers, but grant no permissions to service accounts outside the kube-system namespace (beyond discovery permissions given to all authenticated users).

There are [different ServiceAccount permission approaches](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#service-account-permissions), but we will only go over two:

- Grant a role to an application-specific service account (**best practice**)
  - requires the `serviceAccountName` specified in the pod spec, and for the ServiceAccount to have been created
- Grant a role to the `default` service account in a namespace
  - permissions given to the `default` service account are available to any pod in the namespace that does not specify a `serviceAccountName`. **This is a security concern in live environments without RBAC**

```sh
# create a service account imperatively
kubectl create service account $SERVICE_ACCOUNT_NAME
# assign service account to a deployment
kubectl set serviceaccount deploy $DEPLOYMENT_NAME $SERVICE_ACCOUNT_NAME
# create a role that allows users to perform get, watch and list on pods, see `kubectl create role -h`
kubectl create role $ROLE_NAME --verb=get --verb=list --verb=watch --resource=pods
# grant permissions in a Role to a user within a namespace
kubectl create rolebinding $ROLE_BINDING_NAME --role=$ROLE_NAME --user=$USER --namespace=$NAMESPACE
# grant permissions in a ClusterRole to a user within a namespace
kubectl create rolebinding $ROLE_BINDING_NAME --clusterrole=$CLUSTERROLE_NAME --user=$USER --namespace=$NAMESPACE
# grant permissions in a ClusterRole to a user across the entire cluster
kubectl create clusterrolebinding $CLUSTERROLE_BINDING_NAME --clusterrole=$CLUSTERROLE_NAME --user=$USER
# grant permissions in a ClusterRole to an application-specific service account within a namespace
kubectl create rolebinding $ROLE_BINDING_NAME --clusterrole=$CLUSTERROLE_NAME --serviceaccount=$NAMESPACE:$SERVICE_ACCOUNT_NAME --namespace=$NAMESPACE
# grant permissions in a ClusterRole to the "default" service account within a namespace
kubectl create rolebinding $ROLE_BINDING_NAME --clusterrole=$CLUSTERROLE_NAME --serviceaccount=$NAMESPACE:default --namespace=$NAMESPACE
```

### Lab 12.4. Exploring RBAC

In [lab 12.3](#lab-123-accessing-the-api-from-a-pod-without-kubectl) we were unable to access the PodList API at `kubernetes.default.svc/api/v1/namespaces/default/pods`. Lets apply the required permissions to make this work.

1. Create a ServiceAccount and verify
2. Create a Role with permissions to list pods and verify
3. Create a RoleBinding that grants the Role permissions to the ServiceAccount, within the `default` namespace, and verify
4. Create a "naked" Pod bound to the ServiceAccount
5. Connect an interactive shell to the Pod and use `curl` to PodList API
6. Can you access the API to get a specific Pod like the one you're running? Hint: Role permissions
7. Can you use a deployment instead of a "naked" Pod?

<details>
  <summary><b>lab 12.4 steps</b></summary>

```sh
# create service account yaml
kubectl create serviceaccount test-sa --dry-run=client -o yaml > lab12.yaml
echo --- >> lab12.yaml
# create role yaml
kubectl create role test-role --resource=pods --verb=list --dry-run=client -o yaml >> lab12.yaml
echo --- >> lab12.yaml
# create rolebinding yaml
kubectl create rolebinding test-rolebinding --role=test-role --serviceaccount=default:test-sa --namespace=default --dry-run=client -o yaml >> lab12.yaml
echo --- >> lab12.yaml
# create configmap yaml
kubectl create configmap test-cm --from-literal="SA=/var/run/secrets/kubernetes.io/serviceaccount" --dry-run=client -o yaml >> lab12.yaml
echo --- >> lab12.yaml
# create pod yaml
kubectl run test-pod --image=nginx --dry-run=client -o yaml >> lab12.yaml 
# review & edit yaml to add configmap and service account in pod spec, see `https://k8s.io/examples/pods/pod-single-configmap-env-variable.yaml`
nano lab12.yaml
# create all resources
kubectl apply -f lab12.yaml
# verify resources
kubectl get sa test-sa
kubectl describe sa test-sa | less
kubectl get role test-role
kubectl describe role test-role | less
kubectl get rolebinding test-rolebinding
kubectl describe rolebinding test-rolebinding | less
kubectl get configmap test-cm
kubectl describe configmap test-cm | less
kubectl get pod test-pod
kubectl describe pod test-pod | less
# access k8s API from within the pod
kubectl exec -it test-pod -- bash
TOKEN=$(cat $SA/token)
HEADER="Authorization: Bearer $TOKEN"
curl -H $HEADER https://kubernetes.default.svc/api --insecure
curl -H $HEADER https://kubernetes.default.svc/api/v1/namespaces/default/pods --insecure
curl -H $HEADER https://kubernetes.default.svc/api/v1/namespaces/default/pods/$POD_NAME --insecure
curl -H $HEADER https://kubernetes.default.svc/api/v1/namespaces/default/deployments --insecure
exit
# clean up
kubectl delete -f lab12.yaml
```
</details>

## 13. DevOps

### Helm

There are thousands of people and companies packaging their applications for deployment on Kubernetes. A [best practice](https://kubernetes.io/blog/2016/10/helm-charts-making-it-simple-to-package-and-deploy-apps-on-kubernetes/) is to package these applications as [Helm Charts](https://helm.sh/docs/topics/charts/).

[Helm](https://helm.sh/) is a package manager you can install, like [winget](https://docs.microsoft.com/en-us/windows/package-manager/winget/), npm, yum and apt, and Charts are packages stored locally or on remote Helm repositories, like msi, debs and rpms.

```sh
# helm installation steps
VER=$(curl -s https://api.github.com/repos/helm/helm/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//g')
wget https://get.helm.sh/helm-v$VER-linux-amd64.tar.gz # macOS replace with `darwin-amd64`
tar xvf helm-v3.9.3-linux-amd64.tar.gz
sudo install linux-amd64/helm /usr/local/bin
rm helm-v$VER-linux-amd64.tar.gz
helm version
```

### ArtifactHUB

[ArtifactHUB](https://artifacthub.io/) is a Helm Charts registry, like [docker hub](https://hub.docker.com/) or the [npm registry](https://www.npmjs.com/), used to find, install and publish Charts.

```sh
# add a helm repo
helm repo add $RELEASE_NAME $RELEASE_URL
# install helm chart from an added repo
helm install $RELEASE_NAME $RELEASE_NAME/$CHART_NAME
# list helm repo
helm repo list
# search for charts in a repo
helm search repo $RELEASE_NAME
# update helm repos (running update command after adding new repo is good practice)
helm repo update
# list currently installed charts
helm list
# show details of a chart
helm show chart $RELEASE_NAME/$CHART_NAME
helm show all $RELEASE_NAME/$CHART_NAME
# view status of a chart
helm status $CHART_NAME
# delete currently installed charts
helm delete
# uninstall helm chart
helm uninstall $RELEASE_NAME
```

### Lab 13.1. Explore Helm Charts

1. [Install Helm](https://github.com/helm/helm#install)
2. List installed Helm Charts
3. List installed Helm repos
4. Find and install a Chart from [ArtifactHUB](https://artifacthub.io/)
5. View resources created in your cluster by the Helm Chart
6. Update Helm repo
7. List installed Helm Charts
7. List installed Helm repos
8. View details of installed Chart
9. View status of installed Chart
10. Search for available Charts in added Helm repo

### Helm templates

Helm Charts come with preset configuration stored in a YAML file, some of which may not be of use to you. One powerful feature of Helm is the option to customise the base chart configuration before installation.

```sh
# view chart preset configuration
helm show values $RELEASE_NAME/$CHART_NAME | less
# download a copy of a chart configuation file
helm pull $RELEASE_NAME/$CHART_NAME
# verify a chart template
helm template --debug /path/to/template/directory
# install chart from template
helm install -f /path/to/values.yaml {$NAME|--generate-name} /path/to/template/directory

# working with {tar,tar.gz,tgz,etc} archives, see `tar --help`
# extract tar file to current directory, `-x|--extract`, `-v|--verbose, `-f|--file`
tar -xvf file.tgz
# extract tar file to specified directory, `-C|--directory`
tar -xvf file.tgz -C /path/to/directory
# list contents of a tar file, `-t|--list`
tar -tvf file.tar
```

### Lab 13.2. Customising Helm Charts

1. Download a Chart template and extract the content to a specified directory
2. Edit the template and change any value
3. Verify the edited template
4. Install the edited template
5. View resources created by the Helm Chart
6. List installed Charts
7. List installed Helm repos

### Kustomize

[Kustomize](https://github.com/kubernetes-sigs/kustomize) is a Kubernetes standalone tool to customize Kubernetes resources through a [kustomization.yaml file](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kustomization).

> Kustomize is not currently part of the CKAD curriculum but good to know for general DevOps practice.

[Kustomize can manage configuration files in three ways](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#overview-of-kustomize):

- generating resources from other sources, e.g. generate with [`secretGenerator`](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kustomize/#create-the-kustomization-file) and [`configMapGenerator`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-a-configmap-from-generator)
   <details>
     <summary>configmap generator example</summary>

   ```sh
   cat <<EOF >./kustomization.yaml
   configMapGenerator:
   - name: example-configmap-2
     literals:
     - FOO=Bar
   EOF
   ```
   </details>
- composing and customising collections of resources, e.g. composing two resources together or adding a patch, see example:
   <details>
     <summary>composing & customising example</summary>

   ```sh
   cat <<EOF >./kustomization.yaml
   resources:
   - deployment.yaml # uses 1 replica
   - service.yaml
   patchesStrategicMerge:
   - patch.yaml # change Deployment to 3 replicas
   EOF
   ```
   </details>
- setting cross-cutting fields for resources, e.g. setting same namespaces, name prefix/suffix, labels or annotations to all resources
   <details>
     <summary>cross-cutting fields example</summary>

   ```sh
   cat <<EOF >./kustomization.yaml
   namespace: my-namespace
   namePrefix: dev-
   nameSuffix: "-001"
   commonLabels:
     app: bingo
   commonAnnotations:
     oncallPager: 800-555-1212
   resources:
   - deployment.yaml
   - service.yaml
   EOF
   ```
   </details>

```sh
# create resources from a kustomization file
kubectl apply -k /path/to/directory/containing/kustomization.yaml
# view resources found in a directory containing a kustomization file
kubectl kustomize /path/to/directory/containing/kustomization.yaml
# view resources found in a directory containing a kustomization file
kubectl kustomize /path/to/directory/containing/kustomization.yaml
```

We can take advantage of Kustomization's "composing and customising" feature to create deployment pipelines by using a directory layout where multiple [_overlay_](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#overlay) kustomizations ([variants](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#variant)) refer to a [_base_](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#base) kustomization:

<details>
  <summary>pipeline layout example</summary>

```sh
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── dev
    │   ├── kustomization.yaml # `bases: ['../../base']`, `namePrefix: dev-`
    │   └── patch.yaml
    ├── prod
    │   ├── kustomization.yaml # `bases: ['../../base']`, `namePrefix: prod-`
    │   └── patch.yaml
    └── staging
        ├── kustomization.yaml # `bases: ['../../base']`, `namePrefix: staging-`
        └── patch.yaml
```
</details>

### Lab 13.3. Kustomize resources

1. Create a `service.yaml` resource file for a service
2. Create a `deployment.yaml` resource file for an app using the service
3. Create a `kustomization.yaml` file with name prefix/suffix and common labels for both resource files
4. Apply the Kustomization file to create the resources
5. Review resources created and confirm that the prefix/suffix and labels are applied

### Lab 13.4. Blue/Green deployments

[Blue/green deployment](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/#blue-green-deployment) is a update-strategy used to accomplish zero-downtime deployments. The current version application is marked blue and the new version application is marked green. In Kubernetes, blue/green deployment can be easily implemented with Services.

<details>
  <summary><b>blue/green update strategy</b></summary>
  
  ![blue-green update strategy from https://engineering-skcc.github.io/performancetest/Cloud-%ED%99%98%EA%B2%BD-%EC%84%B1%EB%8A%A5%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/](https://user-images.githubusercontent.com/17856665/185770858-83a088c2-4701-4fd6-943e-ebfb020aa498.gif)
</details>

1. Create a webserver application (blue deployment)
   - three replicas
   - use an older version of the image
   - mount the DocumentRoot `index.html` as volume
2. Make the application accessible via an IP
3. Verify created resources and test access with `curl`
4. Create a new application using [1] as base (green deployment)
   - three replicas
   - use a newer version of the image
   - the landing page should look different from the webserver in [1]
   - mount the DocumentRoot `index.html` as volume
5. Verify created resources and test access with `curl`
6. Make green deployment accessible via an IP by replacing the Service for blue deployment
7. Confirm all working okay with `curl`

### Lab 13.5. Canary deployments

Canary deployment is an update strategy where updates are deployed to a subset of users/servers (canary application) for testing prior to full deployment. This is a scenario where Labels are required to distinguish deployments by release or configuration.

<details>
  <summary><b>canary update strategy</b></summary>
  
  ![canary update strategy from https://life.wongnai.com/project-ceylon-iii-argorollouts-55ec70110f0a](https://user-images.githubusercontent.com/17856665/185770905-7c3901ec-f97a-4046-8411-7a722b0601a4.png)
</details>

1. Create a webserver application
   - three replicas
   - selector `updateType: canary`
   - use an older version of the image
   - mount the DocumentRoot `index.html` as volume
2. Make the application accessible via an IP
3. Verify created resources and test access with `curl`
4. Create a new application using [1] as base
   - one replica
   - selector `updateType: canary`
   - use a newer version of the image in [1]
   - the landing page should look different from the webserver in [1]
   - mount the DocumentRoot `index.html` as volume
5. Verify created resources and confirm the Service targets both webservers
6. Run multiple `curl` requests to the IP in [2] and confirm access to both webservers
7. Scale up the new webserver to three replicas and confirm all Pods running
8. Scale down the old webserver to zero and confirm no Pods running

> Scaling down to zero instead of deleting provides an easy option to revert changes when there are issues

### Custom Resource Definition (CRD)

A _Resource_ is an endpoint in the Kubernetes API that stores a collection of [API objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) of a certain kind; for example, the Pods resource contains a collection of Pod objects.

A [_Custom Resource_](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. Many core Kubernetes functions are now built using custom resources, making Kubernetes more modular.

Although, we only focus on one, there are [two ways to add custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#comparing-ease-of-use) to your cluster:

- [CRDs](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#create-a-customresourcedefinition) allows user-defined resources to be added to the cluster. They are simple and can be created without any programming. In reality, [Operators](#operator-pattern) are preferred to CRDs.
- [API Aggregation](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/) requires programming, but allows more control over API behaviors like how data is stored and conversion between API versions.

```sh
# CRD example "resourcedefinition.yaml"
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com # must match `<plural>.<group>` spec fields below
spec:
  group: stable.example.com # REST API: /apis/<group>/<version>
  versions: # list of supported versions
    - name: v1
      served: true # enabled/disabled this version, controls deprecations
      storage: true # one and only one version must be storage version.
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced # or Cluster
  names:
    plural: crontabs # REST API: /apis/<group>/<version>/<plural>
    singular: crontab # used for display and as alias on CLI
    kind: CronTab # CamelCased singular type for resource manifests.
    shortNames:
    - ct # allow `crontab|ct` to match this resource on CLI
```

### Lab 13.6. Custom objects

You can follow the [official CRD tutorial](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/).

1. Create a custom resource from the snippet above
2. Confirm a new API resource added
3. Create a custom object of the custom resource
   ```sh
   apiVersion: "stable.example.com/v1"
   kind: CronTab
   metadata:
     name: my-new-cron-object
   spec:
     cronSpec: "* * * * */5"
     image: my-awesome-cron-image
   ```
4. Review all resources created and confirm the `shortName` works
5. Directly access the Kubernetes REST API and confirm endpoints for:
   - group `/apis/<group>`
   - version `/apis/<group>/<version>`
   - plural `/apis/<group>/<version>/<plural>`
6. Clean up by deleting with the manifest files

### Operator pattern

[Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) are software extensions to Kubernetes that make use of custom resources to manage applications and their components. The operator pattern captures how you can write code to automate a task beyond what Kubernetes itself provides.

Kubernetes' operator pattern concept lets you extend the cluster's behaviour without modifying the code of Kubernetes itself by linking [controllers](https://kubernetes.io/docs/concepts/architecture/controller/) (a non-terminating loop, or control loop, that regulates the cluster to a desired state) to one or more custom resources. Operators are clients of the Kubernetes API that act as controllers for a Custom Resource.

Although, you can write your own operator, majority prefer to find ready-made operators on community websites like [OperatorHub.io](https://operatorhub.io/). Many Kubernetes solutions are provided as operators like [Prometheus](https://prometheus.io/) or Tigera (calico).

> This lab requires the Calico plugin. You will need to delete and start a new cluster if your current one doesn't support Calico

### Lab 13.7. Operators

See the [official Calico install steps](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube).

```sh
# 1. start a new cluster with network `192.168.0.0/16` or `10.10.0.0/16` whichever subnet is free in your network
minikube start --kubernetes-version=1.23.9 --network-plugin=cni --extra-config=kubeadm.pod-network-cidr=10.10.0.0/16
# 2. install tigera calico operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.0/manifests/tigera-operator.yaml
# 3. install custom resource definitions
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.0/manifests/custom-resources.yaml
# 4. view an installation resource
kubectl get installation -o yaml | less
# 5. verify calico installation
watch kubectl get pods -l k8s-app=calico-node -A
# you can use wget to view a file without saving
wget -O- https://url/to/file | less
wget -qO- https://url/to/file | less # quiet mode
```

1. Start a Minikube cluster with the `cni` network plugin and a suitable subnet
2. List existing namespaces
3. Install the Tigera Calico operator
4. Confirm resources added:
   - new API resources for `tigera`
   - new namespaces
   - resources in the new namespaces
5. Review the CRDs manifest file and ensure matching `cidr`, then install
6. Confirm resources added:
   - new `Installation` resource
   - new namespaces
   - resources in the new namespaces (Calico Pods take awhile to enter `Running` status)

### StatefulSets

[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) is similar to Deployments but provides guarantees about the ordering and uniqueness of managed Pods. Unlike Deployments, StatefulSet Pods are not interchangeable: each has a persistent identifier and storage volume that it maintains across any rescheduling.

#### Using StatefulSets

StatefulSets are valuable for applications that require one or more of the following.

- Stable - persistence across Pod (re)scheduling, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

#### Limitations of StatefulSets

- Storage must either be provisioned by a [PersistentVolume Provisioner]() based on StorageClass, or pre-provisioned by an admin
- To ensure data safety, deleting and/or scaling a StatefulSet down will not delete associated volumes
- You are responsible for creating a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to provide network access to the Pods
- To achieve ordered and graceful termination of Pods, scale the StatefulSet down to 0 prior to deletion
- It's possible to get into a broken state that requires [manual repair](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#forced-rollback) when using [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#rolling-updates) with the default [Pod Management Policy](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies) (`OrderedReady`)

### Lab 13.8. Components of a StatefulSet

See the [example manifest](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components)

1. Create a StatefulSet based on the example manifest
2. Verify resources created and compare to a regular deployment
3. Confirm persistent volume claims created

## 14. Troubleshooting

## 15. Exam
