[[_TOC_]]

# CKAD Guide

## 1. Understanding and Using Containers

### Basic commands

```sh
# test
docker run hello-world
# interactive
docker run -it busybox
# detached
docker run -d -p 8080:80 -v /var/www/html:/var/www/html --name myserver -e ENV_VAR=value httpd
# exec
docker exec -it busybox sh
# list running containers
docker ps
# list all containers
docker ps -a
# view container processes (run within container)
ls /proccat /proc/[pid]/cmdline
```

### Tips

- `docker exec` connects to another shell session  
- `exit` exits running container
- `docker run` connects to entrypoint  
- `exit` exits and stops container  
- `ctrl-p ctrl-q` exits running container
- `docker ps` STATUS of `Exited (0)` means exit OK, but Exit STATUS otherthan 0 should be investigated `docker logs`

### Managing containers

```sh
docker {start | stop | restart | kill | rm}
docker ps [-a] # this command shows container IDs
docker inspect [id] | less
docker inspect --format="{{.NetworkSettings.IPAddress}}" containerName
docker inspect --format="{{.State.Pid}}" containerName
ps {aux | fax} | less # this command shows container PIDs
```

### Container images

```sh
docker image ls  # or `docker images`
docker image inspect [imageId]
docker image rm [imageId] # requires `--force` when containers using the image
docker {image | containers | network | volumes}
```

> note commands after imageId/imageName are passed to image/container
> e.g. `docker run -it mysql -e MYSQL_PASSWORD=hello` will not work cos env vars should come before image name

### Container logging

```sh
docker logs [containerName]
```

### Lab 1

1. Install Docker in your environment
2. Run the latest version of any linux distros in a container
3. Start a bash shell connected to the container interactively
4. Explore the `/etc/os-release` file and the kernel version `uname -r`

## 2. Managing Container Images
