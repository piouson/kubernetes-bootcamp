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

> note commands after imageId/imageName are passed to image/container \
> e.g. `docker run -it mysql -e MYSQL_PASSWORD=hello` will not work cos env vars should come before image name like `docker run -it -e MYSQL_PASSWORD=hello mysql`

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

> See Docker's [Deploy on Kubernetes](https://docs.docker.com/desktop/kubernetes/) for more details \
> Note that using Docker Desktop will have network limitations when exposing your applications publicly, see alternative Minikube option below

### Use Minikube

1. Disable Docker Desktop integration with WSL2 Ubuntu
2. Install on WSL2 Ubuntu: [`docker engine`](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script), [`minikube`](https://minikube.sigs.k8s.io/docs/start/), `conntrack` and [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management)
3. [Enable **`systemd`** on WSL2 Ubuntu](https://github.com/DamionGans/ubuntu-wsl2-systemd-script)
3. Start minikube cluster

```sh
# 1. disable docker desktop integration with wsl2 ubuntu
# 2. install docker engine
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo groupadd docker
sudo usermod -aG docker $USER
# logout or start another terminal to update group membership before testing docker below
docker run hello-world
# 3. install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
# 4. install conntrack
sudo apt install conntrack
# 5. install kubectl if required
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
# 6. Enable `systemd` on wsl2
git clone https://github.com/DamionGans/ubuntu-wsl2-systemd-script.git
cd ubuntu-wsl2-systemd-script/
bash ubuntu-wsl2-systemd-script.sh
# logout or start another terminal to enable systemd
systemctl # long output confirms systemd up and running
sudo sysctl fs.protected_regular=0 # required by minikube
# 7. start a minikube cluster with driver=none
sudo minikube start --driver=none
# 8. optional, if [7] gives `cri-dockerd` error, install `GO` and build `cri-dockerd`, then repeat [7]
git clone https://github.com/Mirantis/cri-dockerd.git
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
sudo ./installer_linux
source ~/.bash_profile
cd cri-dockerd
mkdir bin
go get && go build -o bin/cri-dockerd
sudo mkdir -p /usr/local/bin
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
sudo cp -a packaging/systemd/* /etc/systemd/system
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
# 9. optional, if [7] gives `crictl` error, install `crictl` and repeat [7]
VERSION="v1.24.1"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
# 10. change the owner of the .kube and .minikube directories
sudo chown -R $USER $HOME/.kube $HOME/.minikube
# 11. confirm running cluster IP - should be within host (wsl2) subnet
kubectl cluster-info
# 12. optional, if `kubectl cluster-info` gives permission denied error, edit the path, see https://stackoverflow.com/a/73100683/1235675
nano ~/.kube/config # change path "/root/.minikube -> /home/username/.minikube"
# 13. open kubernetes dashboard - visit provided url
minikube dashboard
# 14. test external access to apps by exposing the kubernetes dashboard
kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard
# change Type=ClusterIP -> "Type=NodePort or Type=LoadBalancer" and save
kubectl get service/kubernetes-dashboard -n kubernetes-dashboard
# visit localhost with the NodePort/LoadBalancer port - e.g. `localhost:31234`
```

> See [outdated WSL+Docker+Kubernetes tutorial](https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/) as reference

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

> Manifest fields are **case sensitive**, **always generate** manifest files to avoid typos \
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

> On WSL2, there are network issues preventing access to the Service NodePort, see [#4199](https://github.com/microsoft/WSL/issues/4199#issuecomment-668270398) [#7879](https://github.com/kubernetes/minikube/issues/7879). \
> This can be avoided by following the steps in [cp4 - use minikube](#use-minikube)

### Lab 8. Exposing your application internally

- Create an nginx deployment called `nginx`
- Scale the deployment to 3 replicas
- Confirm all resources created
- Create a service for the deployment on port 80
  - Access the app from a browser `kubectl port-forward service/nginx LOCAL_PORT:80`
- Confirm all resources created, note service TYPE, CLUSTER-IP and PORT
- Run a new Pod to execute the following commands:
  - confirm DNS server IP `cat /etc/resolv.conf`
  - confirm Gateway/Domain server IP `nslookup [deploymentName]`
- View more details of the service, note IPs, Port, TargetPort and Endpoints
- Compare the details of the service in YAML
- Get all endpoints and service
- Access the application via the host
- Change the service type to NodePort of 32000
  - Can you access the app through the NodePort - `$(minikube ip):32000`?

## 9. Ingress

## 10. Storage

## 11. Variables and Secrets

## 12. K8s API

## 13. DevOps

## 14. Troubleshooting

## 15. Exam