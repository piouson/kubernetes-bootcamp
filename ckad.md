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

#### Host tips

- `docker exec` connects to running container - a different shell session  
- `docker run` starts a new container, downloads image if required 
- `docker ps` STATUS of `Exited (0)` means exit OK, but Exit STATUS otherthan 0 should be investigated `docker logs`

#### Container tips

- `exit` exits container
  - container will be stopped if connected to entrypoint
  - container will keep running if connected to a different shell session
- `ctrl-p ctrl-q` exits container without stopping the container

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

### Basic commands

```sh
# to view image layers/history
docker image history [imageId]
# tagging images
docker tag myimage myimage:1.0
# tags can also be used to add repository location
docker tag myimage domain.com/myimage:1.0
```

### Custom Images

#### Dockerfile

Build a new image from a base image

```Dockerfile
FROM ubuntu
MAINTAINER Piouson
RUN apt-get update && \
    apt-get install -y bash nmap iproute2 && \
    apt-get clean
ENTRYPOINT ["/usr/bin/nmap"]
CMD ["-sn", "172.17.0.0/24"]
```

> Build Dockerfile: `docker build -t nmap /path/to/Dockerfile`

```sh
# specify base image
FROM 
# specify ID - optional
LABEL
# specify author(s) - optional
MAINTAINER
# execute commands
RUN
# specify environment variables used by container
ENV
# copy files from project directory to the image
ADD
# copy files from local project directory to the image - ADD is recommended
COPY
# specify username for RUN, CMD and ENTRYPOINT commands
USER
# specify default command, array form is recommended
# commands can be specified in `shell - space separated` form or `exec - as array` form
ENTRYPOINT ["command"] # `/bin/sh -c` is used if not specified, args passed cannot be overwritten, so recommended to use CMD for args instead for flexibility
# specfify arguments to the entrypoint command, array form is recommended
CMD ["arg1", "arg2"] # if used without ENTRYPOINT, args will be passed to `/bin/sh -c`
# specify metadata to determine where image should run
# Each command in Dockerfile creates a new layer in the image
# You can reduce layers by chaining commands together using `&&`
RUN sudo apt update && \
    sudo apt install -y bash nano && \
    sudo apt clean
# build an image with a Dockerfile
docker build -t imageName[:tag] directory # docker build -t myimage .
```

#### Docker commit

This is a way to save changes made to a running container

```sh
# commit container changes
docker commit -m "commit message" -a "author" imageName newImageName
# verify changes committed
docker image ls
# create compressed image file for export
docker save -o imageName.tar imageName
```

### Lab 2

Create a `Dockerfile` based on the following:

- Based on `fedora`
- Contains `ps` and network tools
- Should run `sshd` process

> In most cases, building an image goes beyond a successful build. Some installed packages require additional steps to run containers successfully

## 3. Understanding Kubernetes

[K8s](kubernetes.io) is an open-source system for automating deployment, scaling and containerized applications management, currently owned by CNCF - Linux Foundation. \
Release cycle is 3 months and deprecated features are dropped in 2 release cycles.

### Lab 3.1. Kubernetes in Google Cloud

- Signup and Login to [console.cloud.google.com](https://console.cloud.google.com)
- Use the "Cluster setup guide" to create "My first cluster"
- Connect to the cluster using the "Cloud Shell"
- Run the basic commands

```sh
# help
kubectl --help | less
# get an overview of available resources
kubectl get all
# create application
kubectl create -h
kubectl create deploy myapp --image=nginx --replcias=3
kubectl get all
# list essential API resources
kubectl api-resources
# delete pod, deployment, service
kubectl delete {pod | deploy | svc} [podName | deploymentName | serviceName]
```

### Basic  Kubernetes API resources

- Deployment: represents the application and provides services
- ReplicaSet: manages scalability - array of pods
- Pods: manages containers

### Lab 3.2. Explore Kubernetes API Resources via GCloud

- Use `kubectl create deploy` to run any image, e.g. nginx image
- Get an overview of resources to confirm created resources
- Delete the pod create and confirm pod is recreated by replicaset
- Use `kubectl create api-resources` to get a list of available API resources
- Delete the deployment and confirm deleted resources
- Delete the service and confirm no resources left

> Remember to delete Google cloud cluster to avoid charges

## 4. Minikube Lab Environment

### Setup Minikube

Setup a local Minikube lab environment using Docker and Ubuntu. Use `minikube-docker-wsl2-setup.sh` script below:

```sh
#!/bin/bash
# Windows setup - requires Docker and WSL2

sudo apt-get update

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
####
echo the script is now ready
echo manually run "minikube start --vm-driver=docker" to start minikube and configure kubectl to use minikube 

sudo usermod -aG docker $USER
newgrp docker

minikube start --vm-driver=docker --cni=calico
```

### Manage Minikube

```sh
# show current status
minikube status
# open K8s dashboard in local browser
minikube dashboard
# start, stop, delete cluster
minikube {start | stop | delete}
# show current IP address
minikube ip
# show current version
minikube version
# connect to minikube cluster
minikube ssh
```

### Lab 4.1. Explore Kubernetes API Resources via Minikube

Repeat [Lab 3.2](#lab-3.2.-explore-kubernetes-api-resources-via-gcloud) in Minikube