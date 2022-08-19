[[_TOC_]]

# CKAD Lab Guide

## What you will learn

- kubernetes
- improve linux understanding
- containers

## 0. Requirements

A Linux/Unix environment with Docker installed

- macOS install [Docker Desktop](https://docs.docker.com/desktop/install/mac-install/)
- Windows
  - option 1 [install Docker engine on WSL2](https://docs.docker.com/engine/install/ubuntu/) and complete [post install steps](https://docs.docker.com/engine/install/linux-postinstall/)
  - option 2 [install Docker Desktop with WSL2 backend](https://docs.docker.com/desktop/windows/wsl/)

## 1. Understanding and Using Containers

A [container](https://www.docker.com/resources/what-container/) is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.. A **container-runtime**, which relies on the host kernel, is required to run a container.

[Docker](https://www.docker.com/) is most popular container-runtime and container-solution, but there are other runtimes like [runc](https://github.com/opencontainers/runc#runc), [cri-o](https://cri-o.io/), [containerd](https://containerd.io/), etc, but the only significant container-solutions today are Docker and [Podman](https://podman.io/)

A container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Container images become containers at runtime. 

The [Open Container Initiative (OCI)](https://opencontainers.org/) creates open industry standards around container formats and runtimes.

A container registry is a repository, or collection of repositories, used to store and access container images. Container registries are a big player in cloud-native application development, often as part of DevOps processes.

### Container commands

Run `docker --help` or visit [docker cli reference](https://docs.docker.com/engine/reference/commandline/docker/) for more details.

```sh
# run busybox container, see `docker run --help`
docker run busybox
# run in interactive mode
docker run -it busybox
# run in detached mode
docker run -d busybox
# run a command in a new container
docker run busybox echo "Hello, World!"
# run container with specified name
docker run -d --name webserver httpd
# execute commands in running container, see `docker exec --help`
docker exec -it busybox sh # `sh` runs in a different shell session
# list running containers
docker ps
# list all containers
docker ps -a
# view kernel details from inside a container
uname -r
cat /proc/version
hostnamectl # linux only
# view container processes from inside a container
ls /proc # to find PID
cat /proc/$PID/cmdline
# exit running container
exit # container is stopped if connected to entrypoint
# exit running container without stopping container
ctrl-p ctrl-q
# start, stop, delete containers, see `docker {start|stop|etc} --help`
docker {start|stop|restart|rm} $CONTAINER_NAME_OR_ID
# view container logs, see `docker logs --help`
docker logs $CONTAINER_NAME_OR_ID
# remove all unused data (including dangling images)
docker system prune
# remove all unused data (including unused images, dangling or not, and volumes)
docker system prune --all --volumes
```

> See [possible container statuses](https://www.baeldung.com/ops/docker-container-states#possible-states-of-a-docker-container) to understand more about container states

### Lab 1.1. Hello docker

1. Run `docker info` to confirm docker client and server statuses
2. Run `docker run hello-world`
3. Run `ps aux` to review running processes on your host device

### Lab 1.2. First container interaction

1. Run a `busybox` container in interactive mode `docker run -it busybox`
2. Review the container kernel details
3. Review the running processes in the container and note PID
4. Exit the container
5. List running containers
6. List all containers
7. Repeat [1-4] but exit the container without stopping it
8. Delete the container

> `docker ps` showing STATUS of `Exited (0)` means exit OK, but an Exit STATUS that's not 0 should be investigated `docker logs`

### Lab 1.3. Container arguments

1. Run a `nginx` container
2. List running containers (use another terminal if stuck)
3. Exit the container
4. Run another `nginx` container in interactive mode
5. List running containers (use another terminal if stuck)
6. Exit the container
7. Run a `busybox` container with command `sleep 20` as argument, see `sleep --help`
8. List running containers (use another terminal if stuck)
9. Exit container
10. Run another `busybox` container in detached mode
11. Connect to the container to execute commands
12. Review the running processes and note the PID
13. Exit container
14. List running containers
15. List all containers
16. Delete all containers

> `CTRL+P, CTRL+Q` only works when running a container in interactive mode, see [docker attach details](https://docs.docker.com/engine/reference/commandline/attach/#description)
> The `Entrypoint` of a container allows the container to run as an executable. This is the default command used when no arguments are passed to the container

### Managing containers and images

```sh
# run container with port and mounted volume
docker run -d -p 8080:80 -v ~/html:/usr/local/apache2/htdocs httpd # visit localhost:8080
# inspect container or image, see `docker inspect --help | docker {image|container} --help`
docker inspect [id] | less
docker {container|image} inspect
# format inspect output to view container network information
docker inspect --format="{{.NetworkSettings.IPAddress}}" containerName
# format inspect output to view container status information
docker inspect --format="{{.State.Pid}}" containerName
# manage images, see `docker image --help`
docker image ls  # or `docker images`
docker image inspect [imageId]
docker image rm [imageId] # requires `--force` when containers using the image
docker {image|containers|network|volumes}
```

> note commands after imageId/imageName are passed to image/container \
> e.g. `docker run -it mysql -e MYSQL_PASSWORD=hello` will not work cos env vars should come before image name like `docker run -it -e MYSQL_PASSWORD=hello mysql`

### Lab 1.4. Container ports and IP

1. Run a `nginx` container with name `webserver`
2. Inspect the container
3. Visit `http://$CONTAINER_IP_ADDRESS` in your browser # this may not work depending on your envrionment network settings
4. Run another `nginx` container with name `webserver` and exposed on port 80
5. Visit http://localhost in your browser
6. Delete the containers

### Lab 1.5. Container volumes

1. Create an `html/index.html` file with some content
2. Run any webserver containers on port 80 and mount the `html` folder to the [DocumentRoot](https://serverfault.com/a/588388)
   - option `nginx` DocumentRoot - `/usr/share/nginx/html`
   - option `httpd` DocumentRoot - `/usr/local/apache2/htdocs`
3. Visit http://localhost
4. List running containers
5. List all containers
6. Clean up with [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)

### Lab 1.6. Container environment variables

1. Run a `mysql` container and review the last output
2. Run a `mysql` container again but specify an environment variable to resolve the output message
3. Visit http://localhost
4. List running containers
5. List all containers
6. Clean up with [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)

### Lab 1.7. Container registries

1. Explore [Docker Hub](https://hub.docker.com/) and search for images you've used so far or images/applications you use day-to-day, like databases, environment tools, etc.

> Container images are created with instructions that determine the default container behaviour at runtime. A familiarity with specific images/applications may be required to understand their default behaviours

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
K8s **release cycle is 3 months** and deprecated features are supported for a minimum of 2 release cycles (6 months).

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

> See Docker's [Deploy on Kubernetes](https://docs.docker.com/desktop/kubernetes/) for more details \
> Note that using Docker Desktop will have network limitations when exposing your applications publicly, see alternative Minikube option below

### Use Minikube

1. Disable Docker Desktop integration with WSL2 Ubuntu
2. Install on WSL2 Ubuntu (only tested 18.04 bionic): [`docker engine`](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script), [`minikube`](https://minikube.sigs.k8s.io/docs/start/), `conntrack` and [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management)
3. [Enable **`systemd`** on WSL2 Ubuntu](https://github.com/DamionGans/ubuntu-wsl2-systemd-script)
3. Start minikube cluster

```sh
# 1. disable docker desktop integration with wsl2 ubuntu
# 2a. enable `systemd` on wsl2
git clone https://github.com/DamionGans/ubuntu-wsl2-systemd-script.git
cd ubuntu-wsl2-systemd-script/
sudo bash ubuntu-wsl2-systemd-script.sh --force
cd ../ && rm -rf ubuntu-wsl2-systemd-script/
exec bash
systemctl # long output confirms systemd up and running
sudo sysctl fs.protected_regular=0 # required by minikube
# 3a. uninstall old docker versions
sudo apt-get remove docker docker-engine docker.io containerd runc
# 3b. setup docker repo
sudo apt-get update
sudo apt-get -y install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 3c. install docker engine
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
# 3d. manage docker as non-root user
sudo groupadd docker
sudo usermod -aG docker $USER
# 3e. start another terminal to update group membership before testing docker
docker run hello-world
# 4. install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x ./minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm -rf minikube-linux-amd64 $HOME/.minikube
# 5. install minikube deps - conntrack
sudo apt install conntrack
# 6a. start a minikube cluster
sudo minikube start --driver=none --kubernetes-version=1.23.9
# 6b. change the owner of the .kube and .minikube directories
sudo chown -R $USER $HOME/.kube $HOME/.minikube
# 6c. create alias for `kubectl` in `.bashrc` or your preferred profile file
printf "
# minikube kubectl
alias kubectl='minikube kubectl --'
" >> ~/.bashrc
exec bash
# 6d. confirm running
kubectl version
minikube dashboard
```

> Ref [WSL+Docker+Kubernetes tutorial](https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/)

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
# list addons
minikube addons list
# enable addons
minikube addons enable [addonName]
```

### Lab 4. Explore Kubernetes API resources via Minikube

- Repeat [Lab 3.2](#lab-32-explore-kubernetes-api-resources-via-gcloud) in Minikube
- Add kubectl autocompletion, see `kubectl completion --help`
- Use the Kubernetes Dashboard to deploy a webserver (nginx) app

## 5. Pods

[Pods](https://kubernetes.io/docs/concepts/workloads/pods/) are the smallest deployable units of computing that you can create and manage in Kubernetes.

### Naked Pods

Pods started without a deployment are called "naked" Pods, and since they are not managed by a replicaset, therefore, "naked" Pods: are not rescheduled on failure; not eligible for rolling updates; cannot be scaled; cannot be replaced automatically 

###  Managing Pods

```sh
# run a pod, see `kubectl run --help`
kubectl run [podName] image=[imageName]
# run a pod with custom args, NOTE anything after ` -- ` is processed by the pod not kubectl
kubectl run [podName] --image=[imageName] -- <arg1> <arg2> ... <argN>
# run a pod interactively and delete after task completion
kubectl run -it mypod --image=busybox --rm --restart=Never -- date
# display running pods, see `kubectl get --help`
kubectl get pods
kubectl get pods [podName] -o yaml | less
# show details of pod, see `kubectl describe --help`
kubectl describe pods [podName] | less
```

> When adding commands or arguments to a `kubectl` command, anything after ` -- ` is processed by the pod not by kubectl \
> Note: prepend `https://k8s.io/examples/` to any example files in the official docs to use the file with `kubectl`

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

> Manifest fields are **case sensitive**, **always generate** manifest files to avoid typos \
> `kubectl apply` creates a new resource, or updates existing resource previously created by `kubectl apply`

### Valid reasons for multi-container Pods

**Always create single container Pods!** However, some scenarios may require a [multi-container Pod pattern](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)

- To initialise primary container (init container)
- To enhance primary container, e.g. for logging, monitoring, etc. ([sidecar container](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-1-sidecar-containers))
- To prevent direct access to primary container, e.g. proxy ([ambassador container](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-2-ambassador-containers))
- To match the traffic/data pattern in other applications in the cluster ([adapter container](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-3-adapter-containers))

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

This is similar to defining API routes on a backend application, except that each defined route points to an application/deployment.

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

## 14. Troubleshooting

## 15. Exam