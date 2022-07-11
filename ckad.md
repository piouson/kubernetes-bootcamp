[[_TOC_]]

# CKAD Guide

## 1. Understanding and Using Containers

### Basic commands

```sh
# run commands in new container, see `docker run --help`
docker run hello-world # entrypoint is used when no command specified after image
# interactive
docker run -it busybox
# detached
docker run -d -p 8080:80 -v /var/www/html:/var/www/html --name myserver -e ENV_VAR=value httpd
# run commands in running container, see `docker exec --help`
docker exec -it busybox sh # `sh` runs in a different shell session
# list running containers
docker ps
# list all containers
docker ps -a
# view container processes (run within container)
ls /proccat /proc/[pid]/cmdline
# exit running container
exit # container is stopped if connected to entrypoint
# exit running container without stopping container
ctrl-p ctrl-q
```

> `docker ps` showing STATUS of `Exited (0)` means exit OK, but Exit STATUS otherthan 0 should be investigated `docker logs`

### Managing containers

```sh
# start, stop, delete containers, see `docker {start|stop|etc} --help`
docker {start|stop|restart|kill|rm} [containerName|containerID]
docker ps [-a] # this command shows container IDs
# inspect container or image, see `docker inspect --help | docker {image|container} --help`
docker inspect [id] | less
docker {container|image} inspect
docker inspect --format="{{.NetworkSettings.IPAddress}}" containerName
docker inspect --format="{{.State.Pid}}" containerName
ps {aux|fax} | less # this command shows container PIDs
```

### Container images

```sh
# see `docker image --help`
docker image ls  # or `docker images`
docker image inspect [imageId]
docker image rm [imageId] # requires `--force` when containers using the image
docker {image|containers|network|volumes}
```

> note commands after imageId/imageName are passed to image/container
> e.g. `docker run -it mysql -e MYSQL_PASSWORD=hello` will not work cos env vars should come before image name

### Container logging

```sh
# see `docker logs --help`
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
# to view image layers/history, see `docker image history --help`
docker image history [imageId]
# tagging images, see `docker tag --help`
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
# build an image with a Dockerfile, see `docker build --help`
docker build -t imageName[:tag] directory # docker build -t myimage .
```

#### Docker commit

This is a way to save changes made to a running container

```sh
# commit container changes, see `docker commit --help`
docker commit -m "commit message" -a "author" imageName newImageName
# verify changes committed
docker image ls
# create compressed image file for export, see `docker save --help`
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
kubectl get all, see `kubectl get --help`
# create application
kubectl create -h
kubectl create deploy myapp --image=nginx --replcias=3
kubectl get all
# get complete list of supported API resources
kubectl api-resources
# delete pod, deployment, service, see `kubectl delete --help`
kubectl delete {pod|deploy|service} [podName|deploymentName|serviceName]
```

### Basic Kubernetes API resources

- Deployment: represents the application and provides services
- ReplicaSet: manages scalability - array of pods
- Pods: manages containers (**note that one container per pod is the standard**)

### Lab 3.2. Explore Kubernetes API resources via GCloud

- Use `kubectl create deploy` to run any image, e.g. nginx image
- Get an overview of resources to confirm created resources
- Delete the pod create and confirm pod is recreated by replicaset
- Use `kubectl create api-resources` to get a list of available API resources
- Delete the deployment and confirm deleted resources
- Delete the service and confirm no resources left

> Remember to delete Google cloud cluster to avoid charges

## 4. Kubernetes Lab Environment

Methods on Windows below used Docker Desktop with WSL2 (Ubuntu) engine.

### Use Docker Desktop

<details>
  <summary>Enable Kubernetes in Docker Desktop settings, then apply and restart</summary>
  
  ![image](https://user-images.githubusercontent.com/17856665/178120815-357cea98-ecf1-4536-81af-a614b7b4cf5e.png)
</details>

```sh
# after docker desktop restarts
kubectl config get-contexts # this should list docker-desktop as an option
kubectl config use-context docker-desktop
```

See Docker's [Deploy on Kubernetes](https://docs.docker.com/desktop/kubernetes/) for more details.

### Use Minikube

See `minikube-docker-wsl2-setup.sh` script file below:

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

#### Managing Minikube

```sh
# show current status, see `minikube --help`
minikube status
# open K8s dashboard in local browser
minikube dashboard
# start, stop, delete cluster, see `minikube {start|stop|delete} --help`
minikube {start|stop|delete}
# show current IP address
minikube ip
# show current version
minikube version
# connect to minikube cluster
minikube ssh
```

### Lab 4. Explore Kubernetes API resources via Minikube

- Repeat [Lab 3.2](#lab-32-explore-kubernetes-api-resources-via-gcloud) in Minikube
- Add kubectl autocompletion, see `kubectl completion --help`
- Use the Kubernetes Dashboard to deploy a webserver (nginx) app

## 5. Pods

A Pod is similar to a server, running one or more containers, accessibile by a single IP address. Pods are typically started bia a deployment.

### Naked Pods

Pods started without a deployment are called "naked" Pods, and since they are not managed by a replicaset, therefore, "naked" Pods: are not rescheduled on failure; not eligible for rolling updates; cannot be scaled; cannot be replaced automatically 

###  Managing Pods

```sh
# run a pod, see `kubectl run --help`
kubectl run [podName] image=[imageName]
# run a pod with custom args, NOTE anything after ` -- ` is processed by the pod not kubectl
kubectl run [podName] --image=[imageName] -- <arg1> <arg2> ... <argN>
# display running pods, see `kubectl get --help`
kubectl get pods
kubectl get pods [podName] -o yaml | less
# show details of pod, see `kubectl describe --help`
kubectl describe pods [podName] | less
```

> When adding commands or arguments to a `kubectl` command, anything after ` -- ` is processed by the pod not by kubectl

### Lab 5.1 Creating Pods

- Create an nginx Pod and confirm creation
- Display details of the Pod and review the Node, IP, container start date/time and Events

### Pod manifest file

```sh
# view the essential/top-level fields of a Pod manifest file
kubectl explain pod
# view more details of an `<Object>` field
kubectl explain {pod.metadata|pod.spec|pod.status} | less
kubectl explain --recursive {pod.metadata|pod.spec|pod.status} | less # recursively view the field
# create, replace, delete resource with YAML file
kubectl {create|apply|replace|delete} -f pod.yaml
# create new or update, if exists, resource from YAML file (update works if resource was created by `kubectl apply`)
# generate YAML file
kubectl run mynginx --image=nginx -o yaml --dry-run=client > pod.yaml
```

> Manifest fields are **case sensitive**, **always generate** manifest files to avoid typos
> `kubectl apply` creates a new resource, or updates existing resource previously created by `kubectl apply`

### Valid reasons for multi-container Pods

Always create single container Pods. But some scenarios may require a multi-container Pod:

- To initialise primary container (init container)
- To enhance primary container, e.g. for logging, monitoring, etc. (sidecar container)
- To prevent direct access to primary container, e.g. proxy (ambassador container)
- To match the traffic/data pattern in other applications in the cluster (adapter container)

### Lab 5.2 Using YAML file

- Generate a base nginx Pod YAML file
- Create a Pod using the YAML file, the container should have arguments, see [create Pod with command and args docs](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#define-a-command-and-arguments-when-you-create-a-pod). Run `kubectl describe` immediately after creation to review container state.
- Create a multicontainer Pod with 2 containers sharing a volume using a YAML file, see [Pod with two container docs](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/#creating-a-pod-that-runs-two-containers). Run `kubectl describe` immediately after creation to review container state.
- Create a multicontainer Pod with 1 container and 1 init container using a YAML file, see [init container docs](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use). Run `kubectl describe` immediately after creation to review container state.

### Using namespaces

```sh
# create namespace
kubectl create namespace [namespaceName]
# run command in a specific namespace, add `-n [namespaceName]`
kubectl run mypod --image=imageName -n namespaceName
# view resources in a namespaces
kubectl get [resourceType=pods,etc] -n namespaceName
# view resources in all namespaces
kubectl get [resourceType=pods,etc] --all-namespaces
# view all resources and their namespaces
kubectl get all -A
# set a namespace to be used for all subsequent commands
kubectl config set-context --current --namespace=namespaceName
# view default namespace
kubectl config view --minify | grep namespace:
```

### Lab 5.3 Namespaces

- Create a namespace with a YAML file, see [create namespace docs](https://kubernetes.io/blog/2015/08/using-kubernetes-namespaces-to-manage/#creating-a-new-namespace)
- Create an nginx Pod in the namespace using the YAML file

> You can save the output of `--dry-run=client -o yaml` in a single file for both namespace and Pod creation

## 6. Exploring Pods

### Troubleshooting a Pod 

```sh
# get pod information in readable format
kubectl get pods [podName] -o yaml | less
# get definition of any field from manifest file
kubectl explain pods.field[.field] # e.g. pods.apiVersion, pods.spec.containers, etc
# get detailed information on a pod
kubectl describe pods [podName]
# connect to containers in a pod, add `-c [containerName]` if multiple containers
kubectl exec -it [podName] -- sh
# within a container, you always have access to `sh` and the `proc` filesystem
cd /proc
cat 1/cmdline
# view cluster logs for a pod
kubectl logs [podName]
```

> `OOMKilled` is linux term for terminated due to out-of-memory

### Lab 6.1 Troubleshoot failing Pod

- Create a Pod with mysql image and confirm Pod is in 'Running' state
- Get detailed information on the Pod and review Events (any multiple attempts?), 'State', 'Last State' and their Exit codes.
- Note that States might continue to change for containers in error states due to retries, so review detailed information on the Pod a few times
- Review cluster logs for the Pod
- Apply relevant fixes until you have a mysql Pod in 'Running' state

### Port forwarding

Port forwarding in Kubernetes should only be used for testing purposes.

```sh
# forward port from local host to container, requires `ctrl+c` to exit
kubectl port-forwarding [podName] [hostPort:containerPort]
```

### Lab 6.2. Expose container port

Create a webserve Pod and access the webpage from host device using port forwarding

### Security context

See [security context docs](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod) for sample YAML file.

```sh
# show details of pod-level security context
kubectl explain pod.spec.securityContext | less
# show details of container-level security context
kubectl explain pod.spec.containers.securityContext | less
```

### Lab 6.3. RBAC
runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000

- Create a Pod that runs busybox as user ID `1000`, group ID `3000` and filesystem group ID `2000`, and disable container privilege escalation. Connect to container and elevate access with `sudo`
- Create a Pod that runs nginx as Non Root. Connect to the container.

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