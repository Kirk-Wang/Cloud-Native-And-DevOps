# Cloud-Native-And-DevOps

### @@@@🤔🤔🤔🤔🤔🤔🤔🤔🤔🤔🤔🤔🤔@@@@
* [CNCF Cloud Native Interactive Landscape](http://landscape.cncf.io/)
* [Development Containers](https://bitnami.com/containers)
* [Turnkey * Containers](https://bitnami.com/containers#turnkey-containers)
* [k8s-for-docker-desktop](https://github.com/AliyunContainerService/k8s-for-docker-desktop)

## Kubernetes features (1.16)

![k8s-arch](./k8s-arch.png)

### What and why of orchestration
* There are many computing orchestrators
* They make decisions about when and where to "do work"
* We've done this since the dawn of computing:Mainframe schedulers, Puppet, Terraform, AWS, Mesos, Hadoop, etc.
* Since 2014 we've had resurgence of new orchestration projects because:
  1. Popularity of distributed computing
  2. Docker containers as a app package and isolated runtime
* We needed "many servers to act like one, and run many containers"
* An the Container Orchestrator was born
* Many open source projects have been created in the last 5 years to:
  * Schedule running of containers on servers
  * Dispatch them across many nodes
  * Monitor and react to container and server health
  * Provide storage, networking, proxy, security, and logging features
  * Do all this in a declarative way, rather than imperative
  * Provide API's to allow extensibility and management

### Major container orchestration projects
* Kubernetes, aka K8s
* Docker Swarm(and Swarm classic)
* Apache Mesos/Marathon
* Cloud Foundry
* Amazon ECS(not OSS, AWS-only)
* HashiCorp Nomad
* Many of these tools run on top of Docker Engine
* Kubernetes is the one orchestator with many distributions

### Kubernetes distributions
* Kubernetes "vanilla upstream"(not a distribution)
* Cloud-Managed distros: AKS, GKE, EKS,DOK...
* Self-Managed distros: RedHat OpenShift, Docker Enterprise, Rancher, Canonical Charmed, openSUSE Kubic...
* Vanilla installers: Kubeadm, kops, kubicorn...
* Local dev/test: Docker Desktop, minikube, microK8s
* CI testing:kind
* Special builds: Rancher K3s
* And Many, many more..(86 as of June 2019)

### Kubernetes concepts
* Kubernetes is a container management system
* It runs and manages containerized applications on a cluster(one or more servers)
* Often this is simply called "container orchestration"
* Sometimes shortened to Kube or K8s("Kay-eights" or "Kates")

### Basic things we can ask Kubernetes to do
* Start 5 containers using image `atseashop/api:v1.3`
* Place an internal load balancer in front of these containers
* Start 10 containers using image `atseashop/webfront:v1.3`
* Place a public load balancer in front of these containers
* It's Black Friday(or Christmas), traffic spikes, grow our cluster and add containers
* New release!Replace my containers with the new image `atseashop/webfront:v1.4`
* Keep processing requests during the upgrade;update my containers one at a time

### Other things that Kubernetes can do for us
* Basic autoscaling
* Blue/green deployment, canary deployment
* Long running services, but also batch(one-off) and CRON-like jobs
* Overcommit our cluster and evict low-priority jobs
* Run services with stateful data(database etc.)
* Fine-grained access control defining what can be done whom on which resources
* Integrating third party services(service catalog)
* Automating complex tasks(operators)

### Kubernetes architecture
* Ha ha ha ha
* OK, I was trying to scare you, it's much simpler than that ❤️

### Kubernetes architecture: the nodes
* The nodes executing our containers run a collection of services:
  * a container Engine(typically Docker)
  * kubelet(the "node agent")
  * kube-proxy(a necessary but not sufficient network component)
* Nodes were formerly called "minions"
  * (You might see that word in older articles or documentation)

### Kubernetes architecture: the control plane
* The Kubernetes logic(its "brains") is a collection of services:
  * the API server(our point of entry to everything!)
  * core services like the scheduler and controller manager
  * `etcd` (a highly available key/value store; the "database" of Kubernetes)
* Together, these services form the control plane of our cluster
* The control plane is also called the "master"

### Running the control plane on special nodes
* It is common to reserve a dedicated node for the control plane
  * (Except for single-node development clusters, like when using minikube)
* This node is then called a "master"
  * (Yes,this is ambiguous:is the "master" a node, or the whole control plane?)
* Normal applications are restricted from running on this node
  * (By using a mechanism called "taints")
* When high availability is required, each service of the control plane must be resilient
* The control plane is then replicated on multiple nodes
  * (This is sometimes called a "multi-master" setup)

### Running the control plane outside containers
* The services of the control plane can run in or out of containers
* For instance: since `etcd` is a critical service, some people deploy it directly on a dedicated cluster(without containers)
  * (This is illustrated on the first "super complicated" schema)
* In some hosted Kubernetes offerings(e.g. AKS, GKE, EKS), the control plane is invisible
  * (We only "see" a Kubernetes API endpoint)
* In that case, there is no "master node"

`For this reason, it is more accurate to say "control plane" ranther than "master"`

### Do we need to run Docker at all?
No!
* By default, Kubernetes uses the Docker Engine to run containers
* Or leverage other pluggable runtimes through the Container Runtime Interface
* We could also use rkt("Rocket") from CoreOS(deprecated)
* containerd: maintained by Docker, IBM, and community
* Used by Docker Engine, microK8s, K3s, GKE, and standalone; has `ctr` CLI
* CRI-O: maintained by Red Hat, SUSE, and community; based on containerd
* Used by OpenShift and Kubic, version matched to Kubernetes

Yes!
* In this course, we'll run our apps on a single node first
* We may need to build images and ship them around
* We can dot these things without Docker
  * (and get diagnosed with NIH syndrome)
* Docker is still the most stable container engine today
  * (but other options are maturing very quickly)

NIH--> Not Invented Here

* On your development environments, CI pipelines...:
  * Yes, almost certainly
* On our production servers:
  * Yes(today)
  * Probally not(in the future)

More information about CRI `on the Kubernetes blog`

### Interacting with Kubernetes
* We will interact with our Kubernetes cluster through the Kubernetes API
* The Kubernetes API is (mostly) RESTful
* It allows us to create, read, update, delete resources
* A few common resource types are:
  * node (a machine == physical or virtual -- in our cluster)
  * pod (group of containers running together on a node)
  * service (stable network endpoint to connect to one or multiple containers)

### Pods
* Pods are a new abstraction!
* A `pod` can have multiple containers working together
* (But you usually only have on container per pod)
* Pod is our smallest deployable unit; Kubernetes can't mange containers directly
* IP address are associated with `pods`, not with individual containers
* Containers in a pod share `loacalhost`, and can share volumes
* Multiple containers in a pod are deployed together
* In reality, Docker doesn't know a pod, only containers/namespaces/volumes 


### Getting a Kubernetes cluster for learning
* Best: Get a environment locally
  * Docker Desktop(Win/MacOS), minikube(Win Home), or microk8s(Linux)
  * Small setup effort;free;flexible environments
  * Requires 2GB+ of memory
* Good: Setup a cloud Linux host to run microk8s
  * Great if you don't have the local resources to run Kubernetes
  * Small setup effort; only free for a while
  * My $50 DigitalOcean coupon lets you run Kubernetes free for a month
* Last choice: Use a browser-based solution
  * Low setup effort; but host is short-lived and has limited resources
  * Not all hands-on examples will work in the browser sandbox

### Docker Desktop(Windows 10/macOS)
* Docker Desktop(DD) is great for a local dev/test setup
* Requires modern macOS or Windows 10 Pro/Ent/Edu(no Home)
* Requires Hyper-V, and disables VirtualBox

Exercise
* Download Windows or macOS versions and install
* For Windows, ensure you pick "Linux Containers" mode
* Once running, enabled Kubernetes in Settings/Perferences

### Check your connection in a terminal
```sh
 kubectl get nodes
```

### minikube(Windows 10 Home)
* A good local install option if you can't run Docker Desktop
* Inspired by Docker Toolbox
* Will create a local VM and configure latest Kubernetes
* Has lots of other features with it `minikube` CLI
* But, requires spearate install of VirtualBox and kubectl
* May not work with older Windows version(YMMV)

Exercise
* Download and install `VirtualBox`
* Download `kubectl`, and add to $PATH
* Download and install `minikube`
* Run `minikube start` to create and run a Kubernetes VM
* Run `minikube stop` when you're done


### microk8s(linux)
* Easy install and management of local Kubernetes
* Made by Canonical(Ubuntu).Installs using `snap`.Works nearly everywhere
* Has lots of other features with its `microk8s` CLI
* But, requires you install snap if not on Ubuntu
* Runs on containerd rather than Docker, no biggie
* Needs alias setup for `microk8s.kubectl`

Exercise
* Install `microk8s`, change group permissions, then set alias in bashrc
```
sudo apt install snapd
sudo snap install microk8s --classic
microk8s.kubectl
sudo usermod -a -G microk8s <username>
echo "alias kubectl='microk8s.kubectl'" >> ~/.bashrc
```

### Web-based options
Last choice: Use a browser-based solution
* Low setup effort; but host is short-lived and has limited resources
* Services are not always working right, and may not be up to date
* Not all hands-on examples will work in the browser sandbox

Exercise
* Use a prebuilt Kubernetes server at `Katacoda`
* Or setup a Kubernetes node at `play-with-k8s.com`
* Maybe try the latest OpenShift at `learn.openshift.com`
* See if instruqt works for `a Kubernetes playground`

### shpod: For a consistent Kubernetes experience...
* You can use `shpod` for examples
* `shpod` provides a shell running in a pod on the cluster
* It comes with many tools pre-installed(helm, stern, curl, jq...)
* These tools are used in many exercises in these slides
* `shpod` also gives you shell completion and a fancy prompt
* Create it with `kubectl apply -f https://k8smastery.com/shpod.yaml`
* Attach to shell with `kubectl attach --namespace=shpod -ti shpod`
* After finishing course `kubectl delete -f https://k8smastery.com/shpod.yaml`

### First contact with `kubectl`
* `kubectl` is(almost) the only tool we'll need to talk to Kubernetes
* It is a rich CLI tool around the Kubernetes API
  * (Everything you can do with `kubectl`, you can do directly with the API)
* On our machines, there is a `~/.kube/config` file with:
  * the Kubernetes API address
  * the path to our TLS certificates used to authenticate
* You can also use the `--kubeconfig` flag to pass a config file
* Or directly `--server`, `--user`, etc.
* `kubectl` can be pronounced "Cube C T L", "Cube cuttle", "Cube cuddle"...
* I'll be using the official name "Cube Control"

### kubectl get
* Let's look at our `Node` resources with `kubectl get`!

Exercise
* Look at the composition of our cluster:
```sh
kubectl get node
```
* These commands are equivalent:
```sh
kubectl get no
kubectl get node
kubectl get nodes
```

### Obtaining machine-readable output
* `kubectl get` con output JSON, YAML, or be directly formatted

Exercise
* Give us more info about the nodes:
```sh
kubectl get nodes -o wide
```
* Let's have some YAML:
```sh
kubectl get no -o yaml
```
See that `kind: List` at the end? It's the type of our result!

### (Ab)using `kubectl` and `jq`
* It's super easy to build custom reports

Exercise
* Show the capacity of all our nodes as a stream of JSON objects:
```sh
kubectl get nodes -o json | jq ".items[] | {name:.metadata.name} + .status.capacity"
```

### Kubectl describe
* `kubectl describe` needs a resource type and (optionally) a resource name
* It is possible to provide a resource name prefix(all matching objects will be displayed)
* `kubectl describe` will retrieve some extra information about the resource

Exercise
* Look at the information available for your node name with one of the following:
```sh
kubectl describe node/<node>
kubectl describe node <node>

kubectl describe node/docker-desktop
```
(We should notice a bunch of control plane pods.)

### Exploring types and definitions
* We can list all available resource types by running `kubectl api-resources`
  * (in Kubernetes 1.10 and prior, this command used to be `kubectl get`)
* We can view the definition for a resource type with:
```sh
kubectl explain type
```
* We can view the definition of a field in a resource, for instance:
```sh
kubectl explain node.spec
```
* Or get the list of all fields and sub-fields:
```sh
kubectl explain node --recursive
```
e.g.
```sh
kubectl explain node
kubectl explain node.spec
```

### Introspection vs. documentation
* We can access the same information by reading the `API documentation`
* The API documentation is usually easier to read, but:
  * it won't show custom types(like Custom Resource Definitions)
  * We need to make sure that we look at the correct version
* `kubectl api-resources` and `kubectl explain` perform introspection
  * (they communicate with the API server and obtain the exact type definitions)

### Type names
* The most common resource names have three forms:
  * singular(e.g. `node`, `service`, `deployment`)
  * plural(e.g. `nodes`, `services`, `deployments`)
  * short(e.g. `no`, `svc`, `deploy`)
* Some resources do not have a short name
* `Endpoints` only have a plural form
  * (because even a single `Endpoints` resource is actually a list of endpoints)

### More `get` commands: Services
* A service is a stable endpoint to connect to "something"
  * (in the initial proposal, they were called "portals")

Exercise
* List the services on our cluster with one of these commands:
```
kubectl get services
kubectl get svc
```
There is already one service on our cluster: the Kubernetes API itself.

### More `get` commands: Listing running containers
* Containers are manipulated through pods
* A pod is a group of containers:
  * running together (on the same node)
  * sharing resources(RAM, CPU; but also network, volumes)

Exercise
* List pods on our cluster:
```sh
kubectl get pods
```

### Namespaces
* Namespaces allow us to segregate resources

Exercise
* List the namespaces on our cluster with one of these commands:
```sh
kubectl get namespaces
kubectl get namespace
kubectl get ns
```

You know what...This `kube-system` thing looks suspicious

In fact, I'm pretty sure it showed up earlier, when we did:
```sh
kubectl describe node <node-name>
```

### Accessing namespaces
* By default, `kubectl` uses the `default` namespace
* We can see resources in all namespaces with `--all-namespaces`

Exercise
* List the pods in all namespaces:
```sh
kubectl get pods --all-namespaces
```
* Since Kubernetes 1.14, we can also use `-A` as a shorter version:
```sh
kubectl get pods -A
```
Here are our system pods!

### What are all these control plane pods?
* `etcd` is our etcd server
* `kube-apiserver` is the API server
* `kube-controller-manager` and `kube-scheduler` are other control plane components
* `coredns` provides DNS-based service discovery(replacing kube-dns as of 1.11)
* `kube-proxy` is the(per-node) component managing port mappings and such
* `<net name>` is the optional(per-node) component managing the network overlay
* the `READY` column indicates the number of conatainers in each pod
* Note: this only shows containers, you won't see host svcs(e.g. microk8s)
* Also Note: you may see different namespaces depending on setup


### Scoping another namespace
* We can also look at a different namespace(other than `default`)

Exercise
* List only the pods in the `kube-system` namespace:
```sh
kubectl get pods --namespace=kube-system
kubectl get pods -n kube-system
```

### Namespaces and other `kubectl` commands
* We can use `-n/--namespace` with almost every `kubectl` command
* Example:
  * `kubectl create --namespace=X` to create something in namespace X
* We can use `-A/--all-namespaces` with most commands that manipulate multiple objects
* Examples:
  * `kubectl delete` can delete resources across multiple namespaces
  * `kubectl label` can add/remove/update labels across multiple namespaces

### What about `kube-public`?
Exercise
* List the pods in the `kube-public` namespace:
```sh
kubectl -n kube-public get pods
```

Nothing!

`kube-public` is created by our installer & `used for security bootstrapping`.

### Exploring `kube-public`
* The only interesting object in `kube-public` is a ConfigMap named `cluster-info`

Exercise
* List ConfigMap objects:
```sh
kubectl -n kube-public get configmaps
```
* inspect `cluster-info`:
```sh
kubectl -n kube-public get configmap cluster-info -o yaml
```
Note the `selfLink` URI:`/api/v1/namespaces/kube-public/configmaps/cluster-info`

We can use that (later in `kubectl context` lectures)!

```sh
kubectl -n kube-public get configmaps
kubectl -n kube-public get configmap cluster-info -o yaml
```

### What about `kube-node-lease`?
* Starting with kubernetes 1.14, there is `kube-node-lease` namespace
  * (or in Kubernetes 1.13 if the NodeLease feature gate is enabled)
* That namespace contains one Lease object per node
* Node leases are a new way to implement node heartbeats
  * (i.e. node regularly pinging the control plane to say "I'm alive!")
* For more details, see `KEP-0009` or the `node controller documentation`

### Running our first containers on Kubernetes
* First things first: we cannot run a container
* We are going to run a pod, and in that pod there will be a single container
* In that container in the pod, we are going to run a simple `ping` command
* Then we are going to start additional copies of the pod

### Starting a simple pod with `kubectl run`
* We need to spcify at least a name and the image we want to use

Exercise
* Let's ping `1.1.1.1`, Cloudflare's `public DNS resolver`:
```sh
kubectl run pingpong --image alpine ping 1.1.1.1
```
(Starting with Kubernetes 1.12, we get a message telling us that `kubectl run` is deprecated. Let's ignore it for now.)

### Behind the scenes of `kubectl run`
* Let's look at the resources that were created by `kubectl run`

Exercise
* List most resource types:
```sh
kubectl get all
```

We should see the following things:
* `deployment.apps/pingpong` (the deployment that we just created)
* `replicaset.apps/pingpong-xxxxxxxxx ` (a replica set created by the deployment)
* `pod/pingpong-xxxxxxxx-yyyyyyy` (a pod created by the replica set)

Note: as of 1.10.1, resource types are displayed in more detail.

### What are these different things?
* A deployment is a high-level construct
  * allows scaling, rolling updates, rollbacks
  * multiple deployments can be used together to implement a `canary deployment`
  * delegates pods management to replica sets
* A replica set is a low-level construct
  * makes sure that a given number of identical pods are running
  * allows scaling
  * rarely used directly
* Note: A replication controller is the (deprecated) predecessor of a replica set


### Our `pingpong` deployment
* `kubectl run` created a deployment, `deployment.apps/pingpong`
```
NAME                       DESIRED     CURRENT      UP-TO-DATE     AVAILABLE  AGE
deployment.apps/pingpong   1           1            1              1          10m 
``` 

* That deployment created a replica set,`replicaset.apps/pingpong-xxxxxxxxxxxxx`
```
NAME                                  DESIRED     CURRENT      READY      AGE
replicaset.apps/pingpong-7c8bbcd9bc   1           1            1          10m 
```

* That replica set created a pod, `pod/pingpong-xxxxxxxxxxxxx-yyyy`
```
NAME                            READY    STATUS      RESTARTS    AGE
pod/pingpong-7c8bbcd9bc-6c9qz   1/1      Running     0           10m
```

* We'll see later how these folks play together for:
  * scaling, high availability, rolling updates

### Viewing container output
* Let's use the `kubectl logs` command
* We will pass either a pod name, or a type/name
  * (E.g. if we specify a deployment or replica set, it will get the first pod in it)
* Unless specified otherwise, it will only show logs of the first container in the pod
  * (Good thing there's only one in ours!)

Exercise
* View the result of our `ping` command:
```
kubectl logs deploy/pingpong
```

### Scaling our application
* We can create additional copies of our container(I mean, our pod) with `kubectl scale`

Exercise
* Scale our `pingpong` deployment:
```sh
kubectl scale deploy/pingpong --replicas 3
```
* Note that this command does exactly the same thing:
```sh
kubectl scale deployment pingpong --replicas 3
```
Note: what if we tried to scale `replicaset.apps/pingpong-xxxxxxx`?

We could! But he deployment would notice it right away, and scale back to the ...

```sh
kubectl scale deploy/pingpong --replicas 3
```

Streaming logs in real time
* Just like `docker logs`, `kuberctl logs` supports conveninent options:
  * `-f/--follow` to stream logs in real time(a la `tail -f`)
  * `--tail` to indicate how many lines you want to see(from the end)
  * `--since` to get logs only after a given timestramp

Exercise
* View the latest logs of our `ping` command:
```sh
kubectl logs deploy/pingpong --tail 1 --follow
```
* Leave that command running, so that we can keep an eye on these logs

### Streaming logs of multiple pods
* What happens if we restart `kubctl logs`?

Exercise
* interrupt `kubectl logs` (with Ctrl-C)
* Restart it:
```sh
kubectl logs deploy/pingpong --tail 1 --follow
```

`kubectl logs` will warn us that multiple pods were found, and that it's showing us only one of them.

Let's leave `kubectl logs` running while we keep exploring.

### Resilience
* The deployment `pingpong` watches its `replica set`
* The replica set ensures that the right number of pods are running
* What happens if pods disappear?

Exercise
* In a separate window, watch the list of pods:
```sh
watch kubectl get pods
```
* Destroy the pod currently shown by `kubectl logs`:
```sh
kubectl delete pod pingpong-xxxxxxxxxxxxx-yyyyyy
```

e.g.
```sh
watch kubectl get pods
kubectl delete pod pingpong-xxxxxxxxxxxxx-yyyyyy
```

### What happened?
* `kubectl delete pod` terminates the pod gracefully
  * (sending it the TERM signal and waiting for it to shutdown)
* As soon as the pod is in "Terminating" state, the Replica Set replaces it
* But we can still see the output of the "Terminating" pod in `kubectl logs`
* Until 30 seconds later, when the grace period expires
* The pod is then killed, and `kubectl logs` exits

### What if we wanted something different?
* What if we wanted to start a "one-shot" container that doesn't get restarted?
* We could use `kubectl run --restart=OnFailure` or `kubectl run --restart=Never`
* These commands would create jobs or pods instead of deployments
* Under the hood, `kubectl run` invokes "generators" to create resource descriptions
* We could also write these resource descriptions ourselves(typically in YAML),
  and create them on the cluster with `kubectl apply -f`(discussed later)
* With `kubectl run --schedule=...`, we can also create cronjobs.

### Scheduling periodic background work
* A Cron Job is a job that will be executed at specific intervals
  * (the name comes from the traditional cronjobs executed by the UNIX crond)
* It requires a schedule,represented as five space-separated fields:
  * minute[0,59]
  * hour[0,23]
  * day of the month[1,31]
  * month of the year[1,12]
  * day of the week([0,6] with 0=Sunday)
* `*` means "all vaild values";`/N` means "every N"
* Example: `*/3 * * * *` means "every three minutes"

### Creating a Cron Job
* Let's create a simple job to be executed every three minutes
* Cron Jobs need to terminate, otherwise they'd run forever

Exercise
* Create the Cron Job:
```sh
kubectl run --schedule="*/3 * * * *" --restart=OnFailure --image=alpine sleep 10
```
* Check the resource that was created:
```sh
kubectl get cronjobs
```

### Cron Jobs in action
* At the specified schedule, the Cron Job will create a Job
* The Job will create a Pod
* The Job will make sure that the Pod completes
  * (re-creating another on if it fails, for instance if its node fails)

Exercise
* Check the Jobs that are created:
```sh
kubectl get jobs
```
(it will take a few minutes before the first job is scheduled.)


### What about that deprecation warning?

* As we can see from the previous slide, `kubectl run` can do many things
* The exact type of resource created is not obvious
* To make things more explicit, it is better to use `kubectl create`:
  * `kubectl create deployment` to create a deployment
  * `kubectl create job` to create a job
  * `kubectl create cronjob` to run a job periodically
    * (since Kubernetes 1.14)
* Eventually, `kubectl run` will be used only to start one-shot pods
  * (see https://github.com/kubernetes/kubernetes/pull/68132)

### Various ways of creating resources
* `kubectl run`
  * easy way to get started
  * versatile
* `kubectl create <resource>`
  * explicit, but lacks some features
  * can't create a CronJob before Kubernetes 1.14
  * can't pass command-lin arguments to deployments
* `kubectl create -f foo.yaml` or `kubectl apply -f foo.yaml`
  * all features are available
  * requires writing YAML

### Streaming logs of multiple pods
* Can we stream the logs of all our `pingpong` pods?

Exercise
* Combine `-l` and `-f` flags:
```sh
kubectl logs -l run=pingpong --tail 1 -f
```

Note: combining `-l` and `-f` is only possible since Kubernetes 1.14!

Let's try to understand why...


### Streaming logs of many pods
* Let's see what happens if we try to stream the logs for more than 5 pods

Exercise
* Scale up our deployment:
```sh
kubectl scale deployment pingpong --replicas=8
```
* Stream the logs
```sh
kubectl logs -l run=pingpong --tail 1 -f
```

We see a message like the following one:
```
error: your are attempting to follow 8 log streams,
but maximum allowed concurency is 5,
use --max-log-requests to increase the limit
```
e.g.
```sh
kubectl scale deployment pingpong --replicas=8
kubectl get pods
kubectl logs -l run=pingpong --tail 1 -f
# error: you are attempting to follow 8 log streams, but maximum allowed concurency is 5, use --max-log-requests to increase the limit
```
### Why can't we stream the logs of many pods?
* `kubectl` opens one connection to the API server per pod
* For each pod, the API server opens one extra connection to the corresponding kubelet
* If there are 1000 pods in our deployments, that's 1000 inbound + 1000 outbound connections on the API server
* This could easily put a lot of stress on the API server
* Prior Kubernetes 1.14, it was decided to not allow multiple connections
* From Kubernetes 1.14, it is allowed, but limited to 5 connetcions
  * (this can be changed with `--max-log-requests`)
* For more details about the rationale, see `PR #67573`

### Shortcomings of `kubectl logs`
* We don't see which pod sent which log line
* If pods are restarted/replaced, the log stream stops
* If new pods are added, we don't see their logs
* To stream the logs of multiple pods, we need to write a selector
* There are external tools to address these shortcomings
  * (e.g.: Stern)


### Accessing logs from the CLI
* The `kubectl logs` commands has limitaions:
  * it cannot stream logs from multiple pods at a time
  * when showing logs from multiple pods, it mixes them all together
* We are going to see how to do it better


### Doing it manually
* We could(if we were so inclined) write a program or script that would:
  * take a selector as an argument
  * enumerate all pods matching that selector (with `kubectl get -l...`)
  * for one `kubectl logs --follow ...` command per container
  * annotate the logs (the output of each `kubectl logs ....` process) with their origin
  * preserve ordering by using `kubectl logs --timestramps ...` and merge the output
* We could do it, but thankfully, other did it for us already!

### Stern
[Stern](https://github.com/wercker/stern) is an open source project by `Wercker`

From the README:
```
Stern allows you to tail multiple pods on Kubernetes and multiple containers within the pod. Each result is color coded for quicker debugging.

The query is a regular expression so the pod name can easily be filtered and you don't need to specify the exact id (for instance omitting the deployment id). If a pod is deleted it gets removed from tail and if a new pod is added it automatically gets tailed.
```

Exactly what we need!

### Installing Stern
* Run `stern`(without arguments) to check if it's installed
```
$ stern
Tail multiple pods and containers from Kubernetes

Usage:
stern pod-query [flags]
```
* If it is not installed, the easiest method is to download a `binary release`
* The following commands will install Stern on a Linux Intel 64 bit machine:
```sh
sudo curl -L -o /usr/local/bin/stern \
  https://github.com/wercker/stern/releases/download/1.11.0/stern_linux_amd64
sudo chmod +x /usr/local/bin/stern
```
* On OS X, just `brew install stern`

### Using Stern
* There are two ways to specify the pods whose logs we want to see:
  * `-l` followed by a selector expression(like with many `kubectl` commands)
  * with a "pod query," i.e. a regex used to match pod names
* These two ways can be combined if necessary

Exercise
* View the logs for all the pingpong containers:
```sh
stern pingpong
```

### Stern conveninent options
* The `--tail N` flag shows the last `N` lines for each container
  * (instead of showing the logs since the creation of the container)
* The `-t`/`--timestamps` flag shows timestamps
* The `--all-namespaces` flag is self-explanatory

Exercise
* View what's up with the `pingpong` system containers:
```sh
stern --tail 1 --timestamps pingpong
```

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our deployment and cronjob:
```sh
kubectl delete deployment/pingpong cronjob/sleep
```

### 19,000 words
* They say, "a picture is worth one thousand words."
* The following 19 slides show what really happens when we run:
```sh
kubectl run web --image=nginx --replicas=3
```

### Exposing containers
* `kubectl expose` creates a service for existing pods
* A service is a stable address for a pod (or a bunch of pods)
* If we want to connect to our pod(s), we need to create a service
* Once a service is created, CoreDNS will allow us to resolve it by name
  * (i.e. after creating service `hello`, the name `hello` will resolve to something)
* There are different types of services, detailed on the following slides:
  * `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`

### Basic service types
* `ClusterIP`(default type)
  * a virtual IP address is allocated for the service(in an internal, private range)
  * this IP address is reachable only from within the cluster(nodes and pods)
  * our code can connect to the service using the original port number
* `NodePort`
  * a port is allocated for the service(by default, in the 30000-32768 range)
  * that port is made available on all our nodes and anybody can connect to it
  * our code must be changed to connect to that new port number

These service types are always available.

Under the hood:`kube-proxy` is using a userland proxy and a bunch of `iptables` rules.

* `LoadBalancer`
  * an external load balancer is allocated for the service
  * the load balancer is configured accordingly
    * (e.g.: a `NodePort` service is created, and the load balancer sends traffic to that port)
  * available only when the underlying infrastructure provides some "load balancer as as service"
    * (e.g. AWS, Azure, GCE, OpenStack...)
* `ExternalName`
  * the DNS entry managed by CoreDNS will just be a `CNAME` to a provided record
  * no port, no IP address, no nothing else is allocated

### Running containers with open ports
* Since `ping` doesn't have anything to connect to, we'll have to run something else
* We could use the `nginx` offical image, but ...
  * ...we wouldn't be able to tell the backends from each other!
* We are going to use `bretfisher/httpenv`, a tiny HTTP server written in Go
* `bretfisher/httpenv` listens on port 8888
* it serves its environment variables in JSON format
* The environment variables will include `HOSTNAME`, which will be the pod name
  * (and therefore, will be different on each backend)

### Creating a deployment for our HTTP server
* We could do `kubectl run httpenv --image=bretfisher/httpenv` ...
* But since `kubectl run` is changing, let's see how to see `kubectl create` instead

Exercise
* In another window, watch the pods(to see when they are created)
```sh
kubectl get pods -w
```
* Create a deployment for this very lightweight HTTP server:
```sh
kubectl create deployment httpenv --image=bretfisher/httpenv
```
* Scale it to 10 replicas:
```sh
kubectl scale deployment httpenv --replicas=10
```

Exercise
```sh
# t1
kubectl get pods -w
# t2
kubectl create deployment httpenv --image=bretfisher/httpenv
# t1
ctrl+c
kubectl get pods
kubectl get all
clear
kubectl get pods -w
# t2
kubectl scale deployment httpenv --replicas=10
kubectl expose deployment httpenv --port 8888
kubectl get service
```

### Services are layer 4 constructs 
* You can assign IP addresses to services, but they are stll layer 4
  * (i.e. a service is not an IP address; it's an IP address + protocol + port)
* This is caused by the current implementation of `kube-proxy`
  * (it relies on mechanisms that don't support layer 3)
* As a result: you have to indicate the port number for your service
* Running services with arbitrary port (or port ranges) requires hacks
  * (e.g. host networking mode)


### Testing our service
* We will now send a few HTTP requests to our pods

Exercise
* Run shpod if not on Linux host so we can access internal ClusterIP
```sh
kubectl apply -f https://bret.run/shpod.yml
kubectl attach --namespace=shpod -it shpod
```
* Let's obtain the IP address that was allocated for our service, programmatically:
```sh
IP=$(kubectl get svc httpenv -o go-template --template '{{ .spec.clusterIP }}')
echo $IP
```
* Send a few requests:
```sh
curl http://$IP:8888/
```
* Too much output? Filter it with `jq`:
```sh
curl -s http://$IP:8888/ | jq .HOSTNAME
exit
kubectl delete -f https://bret.run/shpod.yml
kubectl delete deployment/httpenv
```

### If we don't need a load balancer
* Sometimes, we want to access our scaled services directly:
  * if we want to save a tiny little bit of latency(typically less than 1ms)
  * if we need to connect over arbitrary ports(instead of a few fixed ones)
  * if we need to communicate over another protocol than UDP or TCP
  * if we want to decide how to balance the requests client-side
  * ...
* In that case, we can use a `headless service`

### Headless services
* A headless service is obtained by setting the `clusterIP` field to `None`
  * (Either with `--cluster-ip=None`, or by providing a custom YAML)
* As a result, the service doesn't have a virtual IP address
* Since there is no virtual IP address, there is no load balancer either
* CoreDNS will return the pods' IP addresses as multiple `A` records
* This gives us an easy way to discover all the replicas for a deployment

### Services and endpoints
* A service has a number of "endpoints"
* Each endpoint is a host + port where the service is available
* The endpoints are maintained and updated automatically by Kubernetes

Exercise
* Check the endpoints that Kubernetes has associated with our `httpenv` service:
```sh
kubectl describe service httpenv
```
* In the output, there will be a line starting with `Endpoints:`.
* That line will list a bunch of addresses in `host:port` format. 

### Viewing endpoint details
* When we have many endpoints, our deploy commands truncate the list
```sh
kubectl get endpoints
```
* If we want to see the full list, we can use a different output:
```sh
kubectl get endpoints httpenv -o yaml
```
* These IP addresses should match the addresses of the corresponding pods:
```sh
kubectl get pods -l app=httpenv -o wide
```

### `endpints` not `endpoint`
* `endpoints` is the only resource that cannot be singular
```sh
kubectl get endpoint
# error: the server doesn't have a resource type "endpoint"
```
* This is because the type itself is plural(unlike every other resource)
* There is no `endpoint` object: `type Endpoints struct`
* The type doesn't represent a single endpoint, but a list of endpoints

### Exposing services to the outside world
* The default type(ClusterIP) only works for internal traffic
* If we want to accept external traffic, we can use one of these:
  * NodePort (expose a service on a TCP port between 30000-32768)
  * LoadBalancer (provision a cloud load balancer for our service)
  * ExternalIP (use one node's external IP address)
  * Ingress(a special mechanism for HTTP services)

We'll see NodePorts and Ingresses more in detail later.

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our httpenv resources:
```sh
kubectl delete deployment/httpenv service/httpenv
```

### Kubernetes network model
* TL,DR:
  * Our cluster(nodes and pods) is one big flat IP network.
* In detail:
  * all nodes must be able to reach each other, without NAT
  * all pods must be able to reach each other, without NAT
  * pods and nodes must be able to reach each other, without NAT
  * each pod is aware of its IP address(no NAT)
  * pod IP addresses are assigned by the network implementation
* Kubernetes doesn't mandate any particular implementaion

### Kubernetes network model:the good
* Everything can reach everything
* No address translation
* No port translation
* No new protocol
* The network implementation can decide how to allocate addresses
* IP addresses don't have to be "portable" from a node to another
  * (For example, We can use a subnet per node and use a simple routed topology)
* The specification is simple enough to allow many various implementaions

### Kubernetes network model:the less good
* Everything can reach everything
  * if you want security, you need to add network policies
  * the network implementation you use needs to support them
* There are literally dozens of implementations out there
  * (15 are listed in the Kubernetes documentaion)
* Pods have level 3(IP) connectivity, but services are level 4(TCP or UDP)
  * (Services map to a single UDP or TCP port; no port ranges or arbitrary IP packets)
* `kube-proxy` is on the data path when connecting to a pod or container,and its not particularly fast(relies on userland proxying or iptables)

### Kubernetes network model:in practice
* The nodes we are using have been set up to use kubenet, Calico, or something else
* Don't worry about the warning about `kube-proxy` performance
* Unless you:
  * routinely saturate 10G network interfaces
  * count packet rates in millions per second
  * run high-traffic VOIP or gaming platforms
  * do weird things that involve millions of simultaneous connections
    * (in which case you're already familiar with kernel tuning)
* if necessary, there are alternatives to `kube-proxy`; e.g. `kube-router`

### The Container Network Interface(CNI)
* Most Kubernetes cluster use CNI "plugins" to implement networking
* When a pod is created, Kubernetes delegates the network setup to these plugins
  * (it can be a single plugin, or a combination of plugins, each doing one task)
* Typically, CNI plugins will:
  * allocate an IP address(by calling an IPAM plugin)
  * add a network interface into the pod's network namespace
  * configure the interface as well as required routes, etc.

### Multiple moving parts
* The "pod-to-pod network" or "pod network"
  * provides communication between pods and nodes
  * is generally implemented with CNI plugins
* The "pod-to-service network"
  * provides internal communication and load balancing
  * is generally implemented with kube-prox(or maybe kube-router)
* Network policies:
  * provide firewalling and isolation
  * can be bundled with the "pod network" or provided by another component

### Even more moving parts
* Inbound traffic can be handled by multiple components:
  * something like kube-proxy or kube-router(for NodePort services)
  * load balancers(ideally, connected to the pod network)
* It is possible to use multiple pod networks in parallel
  * (with "meta-plugins" like CNI-Genie or Multus)
* Some solutions can fill multiple roles
  * (e.g. kube-router can be set up to provide the pod network and/or network polcies and/or replace kube-proxy)

### What's this application?
* It is a DockerCoin miner!
* No,you can't buy coffee with DockerCoins
* How DockerCoins works:
  * generate a few random bytes
  * hash these bytes
  * increment a counter(to keep track of speed)
  * repeat forever!
* DockerCoins is not a cryptocurrency
  * (the only common points are "randomness", "hashing" and "coins" in the name)
* DockerCoins is made of 5 services
  * `rng` = web service generating random bytes
  * `hasher` = web service computing hash of POSTed data
  * `worker` = background process calling `rng` and `hasher`
  * `webui` = web interface to watch progress
  * `redis` = data store(holds a counter updated by `worker`)
* These 5 services are visible in the application's Compose file, dockercoins-compose.yml

### How DockerCoins works
* `worker` invokes web service `rng` to generate random bytes
* `worker` invokes web service `hasher` to hash these bytes
* `worker` does this in an infinite loop
* Every sceond, `worker` updates `redis` to indicate how many loops were done
* `webui` queries `redis`, and computes and exposes "hashing speed" in our browser
(See diagram on next slide!)

### Service discovery in container-land
How does each service find out the address of the other ones?
* We do not hard-code IP addresses in the code
* We do not hard-code FQDNs in the code, either
* We just connect to a service name, and container-magic does the rest
  * (And by container-magic, we mean "a crafty, dynamic, embedded DNS server")

### Example in `worker/worker.py`
```py
redis = Redis("redis")

def get_random_bytes():
    r = requests.get("http://rng/32")
    return r.content

def hash_bytes(data):
    r = requets.post("http://hasher/",
                    data=data,
                    headers={"Content-Type":"application/octet-stream"})
```
(Full source code available `here`)

### DockerCoins at work
* `worker` will log HTTP requests to `rng` and `hasher`
* `rng` and `hasher` will log incoming HTTP requests
* `webui` will give us a graph on coins mined per second

### Check out the app in Docker Compose
* Compose is(still) great for local development
* You can test this app if you have Docker and Compose installed
* If not, remember `play-with-docker.com`

Exercise
* Download the compose file somewhere and run it
```sh
curl -o docker-compose.yml https://k8smastery.com/dockercoins-compose.yml
docker-compose up
```
* View the `webui` on `localhost:8000` or click the `8080` link in PWD

### The reason why this graph is not awesome
* The worker doesn't update the counter after every loop, but up to once per second
* The speed is computed by the browser, checking the counter about once per second
* Between two consecutive updates, the counter will increase either by 4, or by 0
* The perceived speed will therefore be 4 - 4 - 4 - 0 - 4 - 4 - 0 etc.
* What can we conclude from this?
* "I'm clearly incapple of writing good frontend code!"😁

### Stopping the application
* If we interrupt Compose(with `^C`),it will politely ask the Docker Engine to stop the app
* The Docker Engine will send a `TERM` signal to the containers
* If the containers do not exit in a timely manner, the Engine sends a `KILL` signal

Exercise
* Stop the application by hitting `^C`

### Shipping images with a registry
* For development using Docker, it has build, ship, and run features
* Now that we want to run on a cluster, things are different
* Kubernetes doesn't have a build feature built-in
* The way to ship(pull) images to Kubernetes is to use a registry

### How Docker registries work(a reminder)
* What happens when we execute `docker run alpine`?
* If the Engine needs to pull the `alpine` image, it expands it into `library/alpine`
* `library/alpine` is expanded into `index.docker.io/library/alpine`
* The Engine communicates with `index.docker.io` to retrieve `library/alpine:latest`
* To use something else than `index.docker.io`, we specify it in the image name
* Examples:
```sh
docker pull gcr.io/google-containers/alpine-with-bash:1.0
docker build -t registry.mycompany.io:5000/myimage:awesome .
docker push registry.mycompany.io:5000/myimage:awesome
```

### Building and shipping images
* There are many options!
* Manually:
  * build locally(with `docker build` or otherwise)
  * push to the registry
* Automatically:
  * build and test locally
  * when ready, commit and push a code repository
  * the code repository notifies an automated build system
  * that system gets the code, builds it, pushes the image to the registry

### Which registry do we want to use?
* There are SAAS products like Docker Hub, Quay, GitLab...
* Each major cloud provider has an option as well
  * (ACR on Azure, ECR on AWS, GCR on Google Cloud...)
* There are also commercial products to run our own registry
  * (Docker Enterprise DTR, Quay, GitLab, JFrog Artifacotry...)
* And open source options, too!
  * (Quay, Portus, OpenShift OCR, Gitlab, Harbor, Kraken...)
  * (I don't mention Docker Distribution here because it's too basic)
* When picking a registry, pay attention to:
  * Its build system
  * Multi-user auth and mgmt(RBAC)
  * Storage feature(replication, cahing, garbage collection)

### Running DockerCoins on Kubernetes
* Create one deployment for each component
  * (hasher, redis, rng, webui, worker)
* Expose deployments that need to accept connections
  * (hasher, redis, rng, webui)
* For redis, we can use the official redis image
* For the 4 others, we need to build images and push them to some registry

### Using images from the Docker Hub
* For everyone's convenience, we took care of building DockerCoins images
* We pushed these images to the DockerHub, under the `dockercoins` user
* These images are tagged with a version number, v0.1
* The full image names are therefore:
  * `dockercoins/hasher:v0.1`
  * `dockercoins/rng:v0.1`
  * `dockercoins/webui:v0.1`
  * `dockercoins/worker:v0.1`


### Running DockerCoins on Kubernetes
* We can now deploy our code(as well as a redis instance)

Exercise
* Deploy `redis`:
```sh
kubectl create deployment redis --image=redis
```
* Deploy everything else:
```sh
kubectl create deployment hasher --image=dockercoins/hasher:v0.1
kubectl create deployment rng --image=dockercoins/rng:v0.1
kubectl create deployment worker --image=dockercoins/worker:v0.1
kubectl create deployment webui --image=dockercoins/webui:v0.1
```

### Is this working?
* After waiting for the deployment to complete, let's look at the logs!
  * (Hint: use `kubectl get deploy -w` to watch deployment events)

Exercise
* Look at some logs:
```sh
kubectl logs deploy/rng
kubectl logs deploy/worker
kubectl logs deploy/worker --follow
```
🤔`rng` is fine... But not `worker`.

Oh right! We forgot to `expose`

### Connecting containers together
* Three deployments need to be reachable by others:`hasher`, `redis`, `rng`
* `worker` doesn't need to be exposed
* `webui` will be dealt with later

Exercise
* Expose each deployment, specifying the right port:
```sh
kubectl expose deployment redis --port 6379
kubectl expose deployment rng --port 80
kubectl expose deployment hasher --port 80
```

```sh
$ minikube start --vm-driver=hyperkit --registry-mirror=https://registry.docker-cn.com --image-mirror-country=cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

$ minikube start --image-mirror-country=cn --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.6.0.iso --registry-mirror=https://dockerhub.azk8s.cn  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --vm-driver="hyperv" --hyperv-virtual-switch="minikube v switch"  --memory=4096 
```

### Exposing services for external access
* Now we would like to access the Web UI
* We will expose it with a `NodePort`
  * (just like we did for the registry)

Exercise
* Create a `NodePort` service for the Web UI:
```sh
kubectl expose deploy/webui --type=NodePort --port=80
```
* Check the port that was allocated:
```sh
kubectl get svc
```

webui        NodePort    10.96.39.74     <none>        80:31900/TCP   9s

e.g. ->> [http://localhost:31900](http://localhost:31900)

### Scaling our demo app
* Our ultimate goal is to get more DockerCoins
  * (i.e. increase the number of loops per second shown on the web UI)
* Let's look at the `architecture` again:
* We're at 4 hashes a second.Let's ramp this up!
* The loop is done in the worker; perhaps we could try adding more workers?


### Adding another worker
* All we have to do is scale the `worker` Deployment

Exercise
* Open two new terminals to check what's going on with pods and deployments:
```sh
kubectl get pods -w
kubectl get deployments -w
```
* Now, create more `worker` replicas:
```sh
kubectl scale deployment worker --replicas=2
```

### Adding more workers
* If 2 workers give us 2x speed,what about 3 workers?

Exercise
* Scale the `worker` Deployment further:
```sh
kubectl scale deployment worker --replicas=3
```
* The graph in the web UI should go up again.
  * (This is looking great! We're gonna be RICH!)

### Adding even more workers
* Let's see if 10 workers give us 10x speed!

Exercise
* Scale the `worker` Deployment to a bigger number:
```sh
kubectl scale deployment worker --replicas=10
# kubectl scale deployment rng --replicas=2
# kubectl scale deployment hasher --replicas=2
```
The graph will peak at 10 hashes/second.
(We can add as many workers as we want: we will never go past 10 hashes/second.)

### Why are we stuck at 10 hashes per second?
* If this was high-quality, production code, we would have instrumentation
  * (Datadog, Honeycomb, New Relic, statsd, Sumologic,...)
* It's not
* Perhaps we could benchmark our web services?
  * (with tools like `ab`, or even simpler, `httping`)

### Benchmarking our web services
* We want to check `hasher` and `rng`
* We are going to use `httping`
* It's just like `ping`, but using HTTP `GET` requests
  * (it measures how long it takes to perform one `GET` request)
* It's used like this:
```sh
httping [-c count] http://host:port/path
```
* Or even simpler:
```sh
httping ip.ad.dr.ess
```
* We will use `httping` on the ClusterIP addresses of our services

### Obtaining ClusterIP addresses
* We can simply check the output of `kubectl get services`
* Or do it programmatically, as in the example below

Exercise
* Retrieve the IP addresses:
```sh
HASHER=$(kubectl get svc hasher -o go-template={{.spec.clusterIP}})
RNG=$(kubectl get svc rng -o go-template={{.spec.clusterIP}})
```
Now we can access the IP addresses of our services through `$HASHER` and `$RNG`.

### Checking `hasher` and `rng` response times
Exercise
* Remember to use `shpod` on macOS and Windows:
```sh
kubectl apply -f https://bret.run/shpod.yml
kubectl attach --namespace=shpod -ti shpod
```
* Check the response times for both services:
```sh
httping -c 3 $HASHER
httping -c 3 $RNG
```

### Let's draw hasty conclusions
* The bottleneck seems to be `rng`
* What if we don't have enough entropy and can't generate enough random numbers?
* We need to scale out the `rng` service on multiple machines!

Note:this is a fiction!We have enough entropy.But we need a pretext to scale out.
* Oops we only have one node for learning.🤔
* Let's pretend and I'll explain along the way.

(in fact, the code of `rng` uses `/dev/urandom`. which never runs out of entropy......

...and is just as good as `/dev/random`)

### Deploying with YAML
* So far, we created resources with the following commands:
  * `kubectl run`
  * `kubectl create deployment`
  * `kubectl expose`
* We can also create resources directly with YAML manifests

### Kubectl apply vs create
* `kubectl create -f whatever.yaml`
  * creates resources if they don't exist
  * if resources already exist, don't alter them(and display error message)
* `kubectl apply -f whatever.yaml`
  * creates resources if they don't exist
  * if resources already exist,update them(to match the definition provided by the YAML file)
  * stores the manifest as an annotation in the resource

### Creating multiple resources
* The manifest can contain multiple resources separated by `---`
```yml
kind: ...
apiVersion: ...
metadata:
  name: ...
  ...
spec:
  ...
---
kind: ...
apiVersion: ...
metadata:
  name: ...
  ...
spec:
  ...
```

* The manifest can also contain a list of resources
```yml
apiVersion: v1
kind: List
items: 
- kind: ...
  apiVersion: ...
  ...
- kind: ...
  apiVersion: ...
  ...
```

### Deploying DockerCoins with YAML
* We provide a YAML manifest with all the resources for DockerCoins
(Deployments and Services)
* We can use it if we need to deploy or redeploy DockerCoins

Exercise
* Deploy or redeploy DockerCoins:
```sh
kubectl apply -f https://k8smastery.com/dockercoins.yaml
```
* Note the warinings if you already had the resources created
* This is because we didn't use `apply` before
* This is OK for us learning, so ignore the warnings
* Generally in production you want to stick with one method or the other

### The Kubernetes Dashboard
* Kubernetes resources can also be viewed with an official web UI
* That dashboard is usually exposed over HTTPS
(this requires obtaining a proper TLS certificate)
* Dashboard users need to authenticate
* We are going to take a dangerous shortcut

### The insecure method
* We could(and should) use Let's Encrypt...
* ...but we don't want to deal with TLS certificates
* We could(and should) learn how authentication and authorization work...
* ...but we will use a guest account with admin access instead

Yes, this will open our cluster to all kinds of shenanigans.Don't do this at home.

### Running a very insecure dashboard
* We are going to deploy that dashboard with one single command
* This command will create all the necessary resources
(the dashboard itself, the HTTP wrpper, the admin/guest account)
* All these resources are defined in a YAML file
* All we have to do is load that YAML file with `kubctl apply -f`

Exercise
* Create all the dashboard resources, with the following command:
```sh
kubectl apply -f https://k8smastery.com/insecure-dashboard.yaml
kubectl get svc dashboard
```

### Dashboard authentication
* We have three authentication options at this point:
  * token (associated with a role that has appropriate permissions)
  * kubeconfig (e.g. using the `~/.kube/config` file)
  * "skip" (use the dashboard "service account")
* Let's use "skip": we're logged in!

By the way, we just added a backdoor to our Kubernetes cluster!

### Running the Kubernetes Dashboard securely
* The steps that we just showed you are for educational purpose only!
* If you do that on your production cluster, people can and will abuse it
* For an in-depth discussion about securing the dashboard
  * check this execllent post on Heptio's blog
* Minikube/microK8s can be enabled with easy commands
  * `minikube dashboard` and `microk8s.enable dashboard`

### Other dashboards
* Kube Web View
  * read-only dashboard
  * optimized for "troubleshooting and incident response"
  * see vision and goals for details
* Kube Ops View
  * "provides a common operational picture for multiple Kubernetes clusters"
* Your Kubernetes distro comes with one!
* Cloud-provided control-planes often don't come with one.

### Security implications of `kubectl apply`
* When we do `kubectl apply -f <URL>`.we create arbitrary resources
* Resources can be evil;imagine a `deployment` that...
  * starts bitcoin miners on the whole cluster
  * hides in a non-default namespace
  * bing-mounts our nodes' filesystem
  * inserts SSH keys in the root account(on the node)
  * encrypts our data and ransoms it

### `kubectl apply` is the new `curl | sh`
* `curl | sh` is convenient
* It's safe if you use HTTPS URLs from trusted sources
* `kubectl apply -f` is convenient
* It's safe if you use HTTPS URLs from trusted sources
* Example:the official setup instructions for most pod networks

### Daemon sets
* We want to scale `rng` in a way that is different from how we scaled `worker`
* We want one(and exactly one) instance of `rng` per node
* We do not want two instances of `rng` on the same node
* We will do that with a daemon set

### Why not a deployment?
* Can't we just do `kubectl scale deployment rng --replicas=...`?
* Nothing guarantees that the `rng` containers will be distributed evenly
* If we add nodes later,they will not automatically run a copy of `rng`
* If we remove(or reboot) a node, one `rng` container will restart elsewhere
(add we will end up with two instances `rng` on the same node)

### Daemon sets in practice
* Daemon sets are great for cluster-wide, per-node processes:
  * `kube-proxy`
  * CNI network plugins
  * monitoring agents
  * hardware management tools(e.g. SCSI/FC HBA agents)
  * etc.
* They can also be restricted to run only on some nodes

### Creating a daemon set
* Unfortunately, as of Kubernetes 1.15, the CLI cannot create daemon sets
* More precisely: it doesn't have a subcommand to create a daemon set
* But any kind of resource can always be created by providing a YAML description:
```sh
kubectl apply -f foo.yaml
```
* How do we create the YAML file for our daemon set?
  * option 1: read the docs
  * option 2: `vi` our way out of it

### Creating the YAML file for our daemon set
* Let's start with the YAML file for the current `rng` resource
Exercise
* Dump the `rng` resource in YAML:
```sh
kubectl get deploy/rng -o yaml > rng.yml
```
* Edit `rng.yml`

### "Casting" a resource to another
* What if we just changed the `kind` field?(It can't be that easy,right?)

Exercise
* Change `kind: Deployment` to `kind: DaemonSet`
* Save, quit
* Try to create our new resource:
```sh
kubectl apply -f rng.yml
```

### Understanding the problem
* The core of the error is:
```sh
error validating data:
[ValidationError(DaemonSet.spec):
unknown field "replicas" in io.k8s.api.extensions.v1beta1.DaemonSetSpec,
...
```
* Obviously, it doesn't make sense to specify a number of replicas for a daemon set
* Workaround:fix the YAML
  * remove the `replicas` field
  * remove the `strategy` field(which defines the rollout mechanism for a deployment)
  * remove the `progressDeadlineSeconds` field(also used by the rollout mechani)
  * remove the `status: {}` line at the end

### Use the `--force`, Luke
* We could also tell Kubernetes to ignore these errors and try anyway
* The `--force` flag's actual name is `--validate=false`

Exercise
* Try to load our YAML file and ignore errors:
```sh
kubectl apply -f rng.yml --validate=false
```
Wait...Now, can it be that easy?

### Checking what we've done
* Did we transform our `deployment` into a `daemonset`?
Exercise
* Look at the resources that we have now:
```sh
kubectl get all
```

#### `deploy/rng` and `ds/rng`
* You can have different resource types with the same name
(i.e. a deployment and a daemon set both named `rng`)
* We still have the old `rng` deployment
```
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rng         1/1     1            1           20h
```
* But how we have the new `rng` daemon set as well
```
NAME                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/rng   1         1         1       1            1           <none>          7s
```

### Too many pods
* If we check with `kubectl get pods`, we see:
  * one pod for the deployment(named `rng-xxxxxxxxxxx-yyyyy`)
  * one pod per node for the daemon set(named `rng-zzzzz`)
```sh
NAME                             READY   STATUS    RESTARTS   AGE
pod/rng-58596fdbd-lqlsp          1/1     Running   0          20h
pod/rng-qnq4n                    1/1     Running   0          7s
```
The daemon set created one pod per node.

In a multi-node setup, master usually have taints preventing pods from running there.

(To schedule a pod on this node anyway,the pod will require appropriate tolerations)

### Is this working?
* Look at the web UI
* The graph should now go above 10 hashes per second!
* It looks like the newly created pods are serving traffic correctly
* How and why did this happen?
(We didn't do anything special to add them to the `rng` service load balancer!)

### Labels and selectors
* The `rng` service is load balancing requests to a set of pods
* That set of pods is defined by the selector of the `rng` service

Exercise
* Check the selector in the `rng` service definition:
```sh
kubectl describe service rng
```
* The selector is `app=rng`
* It means "all the pods having the label app=rng"
(They can have additional labels as well, that's OK!)

### Selector evaluation
* We can use selectors with many `kubectl` commands
* For instance, with `kubectl get`, `kubectl logs`, `kubectl delete`... and more

Exercise
* Get the list of pods matching selector `app=rng`:
```sh
kubectl get pods -l app=rng
kubectl get pods --selector app=rng
```
But...why do these pods(in particular, the new ones)have this `app=rng` label?

Where do labels come from?
* When we create a deployment with `kubectl create deployment rng`,
this deployment gets the label `app=rng`
* The replica sets created by this deployment also get the label `app=rng`
* The pods created by these replica sets also get the label `app=rng`
* When we created the daemon set from the deployment, we re-used the same spec
* Therefore, the pods created by the daemon set get the same labels
* When we use `kubectl run stuff`, the label is `run=stuff` instead

### Updating load balancer configuration
* We would like to remove a pod from the load balancer
* What would happen if we removed that pod, with `kubectl delete pod...`?
  * It would be re-created immediately(by the replica set ro the daemon set)
* What would happen if we removed the `app=rng` label from that pod?
  * It would also be re-created immediately
  * Why?!?

### Seletors for replia sets and daemon sets
* The "mission" of a replica set is:
  * "Make sure that there is the right number of pods matching this spec!"
* The "mission" of a daemon set is:
  * "Make sure that there is a pod matching this spec on each node!"
* In fact, replica sets adn daemon sets do not check pod specifications
* They merely have a selector, and they look for pods matching that selector
* Yes, we can fool them by manually creating pods with the "right" labels
* Bottom line:if we remove our `app=rng` label...
  *...The pod "disappears" for its parent, which re-creates another pod to replace it
* Since both the `rng` daemon set and the `rng` replica set use `app=rng`...
  * ...Why don't they "find" each other's pods?
* Replica sets have a more specific selector, visible with `kubectl describe`
  * (It looks like `app=rng, pod-template-hash=abcd1234`)
* Daemon sets also have a more specific selector, but it's invisible
  * (It looks like `app=rng, controller-revision-hash=abcd1234`)
* As a result,each controller only "sees" the pods it manages

### Removing a pod from the load balancer
* Currently, the `rng` service is defined by the `app=rng` selector
* The only way to remove a pod is to remove or change the `app` label
* ...But that will cause another pod to be created instead!
* What's the solution?
* We need to change the selector of the `rng` service!
* Let's add another label to that selector(e.g. enabled=yes)

### Complex selectors
* If a selector specifies multiple labels, they are understood as a logical AND
(In other words: the pods must match all the labels)
* Kubernetes has support for advanced, set-based selectors
(But these cannot be used with services, at least not yet!)

### The plan
1. Add the label `enabled=yes` to all our `rng` pods
2. Update the selector for the `rng` service to also include `enabled=yes`
3. Toggle traffic to a pod by manually adding/removing the `enabled` label 
4. Profit!
```
Note: if we swap steps 1 and 2, it will cause a short
 service disruption, becasuse there will be
a period of time during which the service selector
 won't match any pod. During that time,
requests to the service will time out.By doing
 things in the order above, we guarantee that
there won't be any interruption.
```

### Adding labels to pods
* We want to add the label `enabled=yes` to all pods that have `app=rng`
* We could edit each pod one by one with `kubectl edit`...
* ... Or we could use `kubectl label` to label them all
* `kubectl label` can use selectors itself

Exercise
* Add `enabled=yes` to all pods that have `app=rng`
```sh
kubectl label pods -l app=rng enabled=yes
```

### Updating the service selector
* We need to edit the service specification
* Reminder: in the service deinition, we will see `app: rng` in two places
  * the label of the service itself(we don't need to touch that one)
  * the selector of the service(that's the one we want to change)

Exercise
* Update the service to add `enabled: yes` to its selector:
```sh
kubectl edit service rng
```

### When the YAML parser is being too smart
* YAML parsers try to help us:
  * `xyz` is the string `"xyz"`
  * `42` is the integer `42`
  * `yes` is the boolean value `true`
* If we want the string `"42"` or the string `"yes"`, we have to quote them
* So we have to use `enabled: "yes"`

For a good laugh:if we had used "ja", "oui", "si"... as the value,it would have world.

### Updating the service selector, take 2
Exercise
* Update the service to add `enabled: "yes"`  to its selector:
```sh
kubectl edit service rng
```
* This time is should work!
* If we did everything correctly, the web UI shouldn't show any change.

### Updating labels
* We want to disable the pod that was created by the deployment
* All we have to do, is remove the `enabled` label from that pod
* To identify that pod, we can use its name
* ... Or rely on the fact that it's the only one with a `pod-template-hash` label
* Good to know:
  * `kubctl label ... foo=` doesn't remove a label(it sets it to an empty string)
  * to remove label `foo`, use `kubectl label ... foo-`
  * to change an existing label, we would need to add `--overwrite`

### Removing a pod from the load balancer
Exercise
* In one window, check the logs of that pod:
```sh
POD=$(kubectl get pod -l app=rng,pod-template-hash -o name)
kubectl logs --tail 1 --follow $POD

(We should see a steady stream of HTTP logs)
```
* In another window, remove the label from the pod:
```sh
kubectl label pod -l app=rng,pod-template-hash enabled-

(The stream of HTTP logs should stop immediately)
```

There might be a slight change in the web UI(since we removed a bit of capacity from `rng` service).if we remove more pods, the effect should be more visible.

### Updating the daemon set
* If we scale up our cluster by adding new nodes, the daemon set will create more pods
* These pods won't have the `enable=yes` label
* If we want these pods to have that label, we need to edit the daemon set spec
* We can do that with e.g.`kubectl edit daemonset rng`


### Labels and debugging
* When a pod is misbehaving, we can delete it:another one will be recreated
* But we can also change its labels
* It will be removed from the load balancer(it won't receive traffic anymore)
* Another pod will be recreated immediately
* But the problematic pod is still here, and we can inspect and debug it
* We can even re-add it to the rotation if necessary
  * (Very useful to troubleshoot intermittent and elusive bugs)

### Labels and advanced rollout control
* Conversely, we can add pods matching a service's selector
* These pods will then receive requests and serve traffic
* Examples:
  * `one-shot pod with all debug flags enabled, to collect logs`
  * pods created automatically, but added to rotation in a second step
  (by setting their label accordingly)
* This gives us building blocks for canary and blue/green deployments

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our DockerCoin resources(for now):
```sh
kubectl delete -f https://k8smastery.com/dockercoins.yaml
kubectl delete daemonset/rng
```

### Authoring YAML
* To use Kubernetes is to "live in YAML"!
* It's more important to learn the foundations then to memorize all YAML keys(hundreds+)
* There are various ways to generate YAML with Kubernetes, e.g.:
  * `kubectl run`
  * `kubectl create deployment`(and a few other `kubectl create` variants)
  * `kubectl expose`
* These commands use "generators" because the API only accepts YAML(actually JSON)
* Pro: They are easy to use
* Con: They have limits
* When and why do we need to write our own YAML?
* How do we write YAML from scratch?
* And maybe,what is YAML?

### YAML Basics(just in case you need a refresher)
* It's technically a superset of JSON, designed for humans
* JSON was good for machines, but not for humans
* Spaces set the structure. One space off and game over
* Remember spaces not tabs,Ever!
* Two spaces is standard, but four spaces works too
* You don't have to learn all YAML features, but key concepts you need:
  * Key/Value Pairs
  * Array/Lists
  * Dictionary/Maps
* Good online tutorials exist here, here, here, and YouTube here

### Basic parts of any Kubernetes resource manifest
* Can be in YAML or JSON, but YAML is 100%.
* Each file contains one or more manifests
* Each manifest describes an API object(deployment, service, etc.)
* Each manifest needs four parts(root key:values in the file)
```yml
apiVersion:
kind:
metadata:
spec:
```

### A simple Pod in YAML
* This is a single manifest that creates one Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.3
```

### Deployment and Service manifests in one YAML file
```yml
apiVersion: v1
kind: Service
metadata:
  name: mynginx
spec:
  type: NodePort
  ports:
  - port: 80
  selector:
    app: mynginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mynginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynginx
  template:
    metadata:
      labels:
        app: mynginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3
```

### The limits of generate YAML
* Advanced(and even not-so-advanced) features require us to write YAML:
  * pods with multiple containers
  * resource limits
  * healthchecks
  * many other resource options
* Other resource types don't have their own commands!
  * DaemonSets
  * StatefulSets
  * and more!
* How do we access these features?

### We don't have to start from scratch
* Output YAML from existing resources
  * Create a resource(e.g. Deployment)
  * Dump its YAML with `kubectl get -o yaml...`
  * Edit the YAML
  * Use `kubectl apply -f ...` with the YAML file to:
  * update the resource(if it's the same kind)
  * create a new resource(if it's a different kind)
* Or...we have the docs, with good starter YAML
  * StatefulSet, DaemonSet, ConfigMap, and a ton more on Github
* Or...we can use `-o yaml --dry-run`

### Generating YAML without creating resources
* We can use the `-o yaml --dry-run` option combo withe `run` and `create`

Exercise
* Generate the YAML for a Deployment without creating it:
```sh
kubectl create deployment web --image nginx -o yaml --dry-run
``` 
* Generate the YAML for a Namespace without creating it:
```sh
kubectl create namespace awesome-app -o yaml --dry-run
```
* We can clean up the YAML even more if we want
  * (for instance, we can remove the `creationTimestamp` and empty dicts)

### Try `-o yaml --dry-run` with other create commands
```sh
clusterrole         # Create a ClusterRole.
clusterrolebinding  # Create a ClusterRoleBinding for a particular ClusterRole.
configmap           # Create a configmap from a local file, directory or literal
cronjob             # Create a cronjob with the specified name
deployment          # Create a deployment with the specified name
job                 # Create a job with the specified name
namespace           # Create a namespace with the specified name
poddisruptionbudget # Create a pod disruption budget with the specified name
priorityclass       # Create a priorityclass with the specified name
quota               # Create a quota with the specified name
role                # Create a role with single rule
rolebinding         # Create a RoleBinding for a particular Role or ClusterRole
secret              # Create a secret using specified subcommand
service             # Create a service using specified subcommand
serviceaccount      # Create a service account with the specified name
```

* Ensure you use valid `create` commands with required options for each

### Writing YAML from scratch, "YAML The Hard Way"
* Paying homage to Kelsey Hightower's "Kubernetes The Hard Way"
* A reminder about manifest:
  * Each file contains one or more manifests
  * Each manifest describes an API object(deployment, service, etc.)
  * Each manifest needs four parts(root key:values in the file)
```yml
apiVersion: # find with "kubectl api-versions"
kind:       # find with "kubectl api-resources"
metadata:
spec:       # find with "kubectl describe pod"
```
* Those three `kubectl` commands, plus the API docs, is all we'll need

### General workflow of YAML from scrach
* Find the resource `kind` you want to create(`api-resources`)
* Find the latest `apiVersion` your cluster supports for `kind`(`api-versions`)
* Give it a `name` in metadata(minimum)
* Dive into the `spec` of that `kind`
  * `kubectl explain <kind>.spec`
  * `kubectl explain <kind> --recursive`
* Browse the docs `API Reference` for your cluster version to supplement
* Use `--dry-run` and `--server-dry-run` for testing
* `kubectl create` and `delete` until you get it right

```sh
kubectl api-resources
kubectl api-versions
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.volumes
kubectl explain pod.spec --recursive
```

### Advantage of YAML
* Using YAML(instead of `kubectl run/create/etc.` )allows to be declarative
* The YAML describes the desired state of our cluster and applications
* YAML can be stored. versioned, archived(e.g. in git repositories)
* To change resource, change the YAML files(instead of using `kubectl edit/scale/label/etc.`)
* Changes can be reviewed before being applied
(with code reviews, pull requests...)
* This workflow is sometimes called "GitOps"
(there are tools like Weave Flux or GitKube to facilitate it)

### YAML in practice
* Get started with `kubectl run/create/expose/etc.`
* Dump the YAML with `kubectl get -o yaml`
* Tweak that YAML and `kubectl apply` it back
* Store that YAML for reference(for further deployments)
* Feel free to clean up the YAML
  * remove fields you don't know
  * check that it still works!
* `That YAML` will be useful later when using e.g. Kustomize or Helm

### YAML linting and validation
* Use generic linters to check proper YAML formatting
  * yamllint.com
  * codebeautiful.org/yaml-validator
* For humans without kubectl, use a web Kubernetes YAML validator: kubeyaml.com
* In CI, you might use CLI tools
  * YAML linter: `pip install yamllint` github.com/adrienverge/yamllint
  * Kubernetes validator: `kubeval` github.com/instrumenta/kubeval
* We'll learn about Kubernetes cluster-specific validation with kubectl later

### Using server-dry-run and diff
* We already talked about using `--dry-run` for building YAML
* Let's talk more about options for testing YAML
* Including testing against the live cluster API!

### Using `--dry-run` with `kubectl apply`
* The `--dry-run` option can also be used with `kubectl apply`
* However, it can be mileading(it doesn't do a "real" dry run)
* Let's see what happens in the following scenario:
  * generate the YAML for a Deployment
  * tweak the YAML to transform it into a DaemonSet
  * apply that YAML to see what would actually be created

### The limits of `kubectl apply --dry-run`
Exercise
* Generate the YAML for a deployment:
```sh
kubectl create deployment web --image=nginx -o yaml > web.yaml
```
* Change the `kind` in the YAML to make it a `DaemonSet`
* Ask `kubectl` what would be applied:
```sh
kubectl apply -f web.yaml --dry-run --validate=false -o yaml
```
The resulting YAML doesn't represent a valid DaemonSet.

### Server-side dry run
* Since Kubernetes 1.13, we can use server-side dry run and diffs
* Server-side dry run will do all the work, but not persist to etcd
  * (all validation and mutation hooks will be executed)

Exercise
* Try the same YAML files as earlier,with server-side dry run:
```sh
kubectl apply -f web.yaml --server-dry-run --validate=false -o yaml
```
* The resulting YAML doesn't have the `replicas` field anymore.
* Instead, it has the fields expected in a DaemonSet

### Advantages of server-side dry run
* The YAML is verified much more extensively
* The only step that is skipped is "write to etcd"
* YAML that passes server-side dry run should apply successfully
(unless the cluster state changes by the time the YAML is actually applied)
* Validating or mutating hooks that have side effects can also be an issue

### `kubectl diff`
* Kubernetes 1.13 also introduced `kubectl diff`
* `kubectl diff` does a server-side dry run, and shows differences

Exercise
* Try `kubectl diff` on a simple Pod YAML:
```sh
curl -O https://k8smastery.com/just-a-pod.yaml
kubectl apply -f just-a-pod.yaml
# edit the image ta to :1.17
kubectl diff -f just-a-pod.yaml
```
Note: we don't need to specify `--validate=false` here.

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our "hello" pod:
```sh
kubectl delete -f just-a-pod.yaml
```

### Rolling updates
* By default(without rolling updates), when a scaled resource is updated:
  * new pods are created
  * old pods are terminated
  * ...all at the same time
  * if something goes wrong,
* With rolling updates, when a Deployment is updated, it happens progressively
* The Deployment controls multiple ReplicaSets
* Each ReplicaSet is a group of identical Pods
(with the same image, arguments, parameters)
* During the rolling update, we have at least two ReplicaSets:
  * the "new" set(corresponding to the "target" version)
  * at least one "old" set
* We can have multiple "old" sets
(if we start another update before the first one is done)

### Update strategy
* Two parameters determine the pace of the rollout: `maxUnavailable` and `maxSurge`
* They can be specified in absolute number of pods, or percentage of the `replicas` count
* At any given time...
  * there will always be at least `replicas-maxUnavailable` pods available
  * there will never be more than `replicas` + `maxSurge` pods in total
  * there will therefore be up to `maxUnavailable` + `maxSurge` pods being updated
* We have the possibility of rolling back to the previous version
(if the update fails or is unsatisfactory in any way)

### Checking current rollout parameters
* Recall how we build custom reports with `kubectl` and `jq`:

Exercise
* Show the rollout plan for our deployments:
```sh
kubectl apply -f https://k8smastery.com/dockercoins.yaml

kubectl get deploy -o json | jq ".items[] | {name:.metadata.name} + .spec.strategy.rollingUpdate" 
```

### Rolling updates in practice
* As of Kubernetes 1.8, we can do rolling updates with:
 `deployments`, `daemonsets`, `statefulsets`
* Editing one of these resources will automatically result in a rolling update
* Rolling updates can be monitored with the `kubectl rollout` subcommand

### Rolling out the new `worker` service
Exercise
* Let's monitor what's going on by opening a few terminals, and run :
```sh
kubectl get pods -w
kubectl get replicasets -w
kubectl get deployments -w
```
* Update `worker` either with `kubectl edit`, or by running:
```sh
kubectl set image deploy worker worker=dockercoins/worker:v0.2
```
That rollout should be pretty quick. What shows in the web UI?

### Give it some time
* At first, it looks like nothing is happening(the graph remains at the same level)
* According to `kubectl get deploy -w`, the `deployment` was updated really quickly
* But `kubectl get pods -w` tells a different story
* The old `pods` are still here, and they stay in `Terminating` state for a while
* Eventually, they are terminated; and then the graph decreases significantly
* This delay is due to the fact that our worker doesn't handle signals
* Kubernetes sends a "polite" shutdown request to the worker,which ignores it
* After a grace period, Kubernetes gets impatient and kills the container
(The grace period is 30 seconds, but can be changed if needed)

### Rolling out something invalid
* What happens if we make a mistake?

Exercise
* Update `worker` by specifiying a non-existent image:
```sh
kubectl set image deploy worker worker=dockercoins/worker:v0.3
```
* Check what's going on:
```sh
kubectl rollout status deploy worker
```
Our rollout is stuck. However, the app is not dead.
(After a minute, it will stabilize to be 20-25% slower.)
```sh
kubectl get pods
# ErrImagePull
```

### What's going on with our rollout?
* Why is our app a bit slower?
* Because `MaxUnavailable=25%`
  * ...So the rollout terminated 2 replicas out of 10 available
* Okay, but why do we see 5 new replicas being rolled out?
* Because `MaxSurge=25%`
...So in addition to replacing 2 replicas, the rollout is also starting 3 more
* It rounded down the number of MaxUnavailable pods conservatively.
but the total number of pods being rolled out is allowed to be 25+25=50%


### The nitty-gritty details
* We start with 10 pods running for the `worker` deployment
* Current settings: MaxUnavailable=25% and MaxSurge=25%
* When we start the rollout:
  * two replicas are taken down(as per MaxUnavailable=25%)
  * two others are created(with the new version) to replace them
  * three others are created(with the new version) per MaxSurge=25%)
* Now we have 8 replicas up and running, and 5 being depoyed
* Our rollout is stuck at this point!

### Checking the dashboard during the bad rollout
* If you didn't deploy the Kubernetes dashboard earlier, just kip this slide.

Exercise
* Connect to the dashboard that we deployed earlier
* Check that we have failures in Deployments, Pods, and Replica Sets
* Can we see the reason for the failure?

```sh
kubectl describe deploy worker
```

### Recovering from a bad rollout
* We could push some `v0.3` image
(the pod retry logic will eventually catch it and the rollout will proceed)
* Or we could invoke a manual rollback

Exercise
* Cancel the deployment and wait for the dust to settle:
```sh
kubectl rollout undo deploy worker
# deployment.extensions/worker rolled back
kubectl rollout status deploy worker
# deployment "worker" successfully rolled out
```

### Rolling back to an older version
* We reverted to `v0.2`
* But this version still has a performance problem
* How can we get back to the previous version?

### Multiple "undos"
* What happens if we try `kubectl rollout undo` again?

Exercise
* Try it:
```sh
kubectl rollout undo deployment worker
```
* Check the web UI, the list of pods....

🤔That didn't work.

### Multiple "undos" don't work
* If we see successive versions as a stack:
  * `kubectl rollout undo` doesn't "pop" the last element from the stack
  * it copies the N-1th element to the top
* Multiple "undos" just swap back and forth between the last two versions!

Exercise
* Go back to v0.2 again:
```sh
kubectl rollout undo deployment worker
```

### In this specific scenario
* Our version numbers are easy to guess
* What if we had used git hashes?
* What if we had changed other parameters in the Pod spec?

### Listing versions
* We can list successive versions of a Deployment with `kubectl rollout history`

Exercise
* Look at our successive versions:
```sh
kubectl rollout history deployment worker
```
We don't see all revisions.

We might see something like 1, 4, 5.

(Depending on how many "undos" we did before.)

### Explaining deployment revisions
* These revisions correspond to our ReplicaSets
* This information is stored in the ReplicaSet annotations

Exercise
* Check the annotations for our replica sets:
```sh
kubectl describe replicasets -l app=worker | grep -A3 Annotations
```

### What about the missing revisions?
* The missing revisions are stored in another annotation:
```sh
deployment.kubernetes.io/revision-history
```
* These are not shown in `kubectl rollout history`
* We could easily reconstruct the full list with a script
(if we wanted to!)

### Rolling back to an older version
* `kubectl rollout undo` can work with a revision number

Exercise
* Roll back to the "known good" deployment version:
```sh
kubectl rollout undo deployment worker --to-revision=1
```
* Check the web UI or the list of pods

### Changing rollout parameters
* What if we wanted to, all at once:
  * change image to `v0.1`
  * be conservative on availability(always have desired number of available workers)
  * go slow on rollout speed(update only one pod at a time)
  * give some time to our workers to "warm up" before starting more
* The corresponding changes can be expressed in the following YAML snippet:

```yml
spec:
  template:
    spec:
      containers:
      - name: worker
        image: dockercoins/worker:v0.1
  strategy:
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  minReadySeconds: 10
```

### Applying changes through a YAML patch
* We could use `kubectl edit deployment worker`
* But we could also use `kubectl patch` with the exact YAML shown before
* Apply our changes and wait for them to take effect:
```sh
kubectl patch deployment worker -p "
spec:
  template:
    spec:
      containers:
      - name: worker
        image: dockercoins/worker:v0.1
  strategy:
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  minReadySeconds: 10
"
kubectl rollout status deployment worker
kubectl get deploy -o json worker | jq "{name: .metadata.name} + .spec.strategy.rollingUpdate"
```

### Healthchecks
* Healthchecks are key to providing built-in lifecycle automation
* Healthchecks are probes that apply to containers(not to pods)
* Kubernetes will take action on containers that fail healthchecks
* Each container can have three(optional) probes:
  * liveness = is this container dead or alive?(most important probe)
  * readiness = is this container ready to serve traffic?(only needed if a service)
  * startup = is this container still starting up?(alpha in 1.16)
* Different probe handlers  are available(HTTP, TCP, program execution)
* They don't replace a full monitoring solution
* Let's see the difference and how to use them!

### Liveness probe
* Indicates if the container is dead or alive
* A dead container cannot come back to life
* If the liveness probe fails, the container is killed
(to make really sure that it's really dead; no zombies or undeads!)
* What happens next depends on the pod's `restartPolicy`:
  * `Never`: the container is not restarted
  * `OnFailure` or `Always`: the container is restarted

### When to use a liveness probe
* To indicate failures that can't be recovered
  * deadlocks(causing all requests to time out)
  * internal corruption(causing all requests to error)
* Anything where our incident response would be "just restart/reboot it"
 * Do not use liveness probes for problems that can't be fixed by a restart
* Otherwise we just restart our pods for no reason, creating useless load

### Readiness probe
* Indicates if the container is ready to serve traffic
* If a container becomes "unready" it might be ready again soon
* If the readiness probe fails:
  * the container is not killed
  * if the pod is memeber of a service, it is temporarily removed
  * it is re-added as soon as the readiness probe passes again

### When to use a readiness probe
* To indicate failure due to an external cause
 * database is down or unreachable
 * mandatory auth or other backend service unavailable
* To indicate temporary failure or unavailability
  * application can only service N parallel connections
  * runtime is busy doing garbage collection or ini

### Startup probe
* Kubernetes 1.16 introduces a third tye of probe: `startupProbe`
(it is in `alpha` in Kubernetes 1.16)
* It can be used to indicate "container not ready yet"
  * process is still starting
  * loading external data, priming caches
* Before Kubernetes 1.16, we had to use the `initialDelaySeconds` parameter
(available for both liveness and readiness probes)
* `initialDelaySeconds` is a rigid delay(always wait X before running probes)
* `startupProbe` works better when a container start time can vary a lot

### Benefits of using probes
* Rolling updates proceed when containers are actually ready
(as opposed to merely started)
* Containers in a broken state get killed and restarted
(instead of serving errors or timeouts)
* Unavailable backends get removed from load balancer rotation
(thus improving response times across the board)
* If a probe is not defined, it's as if there was an "always successful" probe

### Different types of probe handlers
* HTTP request
  * specify URL of the request(and optional headers)
  * any status code between 200 and 399 indicates success
* TCP connection
  * the probe succeeds if the TCP port is open
* arbitrary exec
  * a command is executed in the container
  * exit status of zero indicates success

### Timing and thresholds
* Probes are executed at intervals of `periodSeconds`(default: 10)
* The timeout for a probe is set with `timeoutSeconds`(default: 1)

If a probe takes longer than that,it is considered as a FAIL
* A probe is considered successful after `successThreshold` successes(default: 1)
* A probe is considered failing after `failureThreshold` failures (default: 3)
* A probe can have an `initialDelaySeconds` parameter(default: 0)
* Kubernetes will wait that amount of time before running the probe for the first time
(this is important to avoid killing services that take a long time to start)

### Example:HTTP probe
Here is a pod template for the `rng` web service of the DockerCoins app:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: rng-with-liveness
spec:
  containers:
  - name: rng
    image: dockercoins/rng:v0.1
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 1
```

If the backend serves an error, or takes longer than 1s, 3 times in a row,it gets killed.

Example: exec probe

Here is a pod template for a Redis server:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-liveness
spec:
  containers:
  - name: redis
    image: redis
    livenessProbe:
      exec:
        command: ["redis-cli", "ping"]
```

If the Redis process becomes unresponsive, it will be killed.

### Should probes check container Dependencies?
* A HTTP/TCP probe can't check an external dependency
* But a HTTP URL could kick off code to validate a remote dependency
* If a web server depends on a database to function, and the database is down: 
  * the web server's liveness probe should succeed
  * the web server's readiness probe should fail
* Same thing for any hard dependency(without which the container can't work)

Do not fail liveness probes for problems that are external to the container

### Healthchecks for workers
(In that context, worker = process that doesn't accept connections)
* Readiness isn't useful(because workers aren't backends for a service)
* Liveness may help us restart a broken worker, but how can we check it?
* Embedding an HTTP server is a(potentially expensive) option
* Using a "lease" file can be relatively easy:
  * touch a file during each iteration of the main loop
  * check the timestamp of that file from an exec probe
* Writing logs(and checking them from the probe) also works

### Questions to ask before adding healthchecks
* Do we want liveness, readiness, both?
(sometimes, we can use the same check, but with different failture thresholds)
* Do we have existing HTTP endpoints that we can use?
* Do we need to add new endpoints, or perhaps use something else?
* Are our healthchecks likely to use resources and/or slow down the app?
* Do they depend on additional services?
(this can be particularly tricky)

### Adding healthchecks to an app
* Let's add healthchecks to DockerCoins!
* We will examine the questions of the previous slide
* Then we will review each component individually to add healthchecks

### Liveness, readiness, or both?
* To answer that question, we need to see the app run for a while
* Do we get temporary, recoverable glitches?
  * -> then sue liveness
* Or do we get hard lock-ups requiring a restart?
  * -> then use liveness
* In the case of DockerCoins, we don't know yet!
* Let's pick liveness

### Do we have HTTP endpoints that we can use？
* Each of the 3 web services(hasher, rng, webui) has a trivial route on `/`
* These routes:
  * don't seem to perform anything complex or expensive
  * don't seem to call other services
* Perfect!
(See next slides for individual details)

hasher.rb
```rb
get '/' do
  "HASHER running on #{Socket.gethostname}\n"
end
```

rng.py
```py
@app.route("/")
def index():
  return "RNG running on {}\n".format(hostname)
```

webui.js
```js
app.get('/', function (req, res) {
  res.redirect('/index.html')
})
```

### Retrieving DockerCoins manifests
* I've split up the previous `dockercoins.yaml` into one-resource-per-file
* This works with the `apply` command, and is easier for humans to manage
* Clone them locally so we can add healthchecks and re-apply

Exercise
* Clone that repository:
```sh
git clone https://github.com/bretfisher/kubercoins
```
* Change directory to the repository:
```sh
cd kubercoins
```

### A simple HTTP liveness probe

This is what our liveness probe should look like:
```yml
containers:
- name: ...
image: ...
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 30
  periodSeconds: 5
```
This will give 30 seconds to the service to start.(Way more than necessary!)

It will run the probe every 5 seconds.

It will use the default timeout(1 second).

It will use the default failure threshold(3 failed attempts = dead).

It will use the default success threshold(1 successful attempt = alive).


### Adding the liveness probe
* Let's add the liveness probe, then deploy DockerCoins
* Remember if you don't have DockerCoins running, this will create
* If you already have DockerCoins running, this will update `rng`

Exercise
* Edit `rng-deployment.yaml` and add the liveness probe
```sh
vim rng-deployment.yaml
```
* Load the YAML for all the resources of DockerCoins
```sh
kubectl apply -f .
```

### Testing the liveness probe
* The rng service needs 100ms to process a request
(because it is single-threaded and sleeps 0.1s in each request)
* The probe timeout is set to 1 second
* If we send more than 10 requests per second per backend,it will break
* Let's generate traffic and see what happens!

Exercise
* Get the ClusterIP address of the rng service:
```sh
kubectl get svc rng
```

### Monitoring the rng service
* Each command below will show us what's happening on a different level

Exercise
* In one window, monitor cluster events:
```sh
kubectl get events -w
```
* In another window, monitor pods status:
```sh
kubectl get pods -w
```

### Generating traffic
* Let's use `ab` (Apache Bench) to send concurrent requests to rng

Exercise
* In yet another window, generate traffic using `shpod`:
```sh
kubectl attach --namespace=shpod -ti shpod
ab -c 10 -n 1000 http://<ClusterIP>/1
```
* Experiment with higher values of `-c` and see what happens

* The `-c` parameter indicates the number of concurrent requests
* The final `/1` is important to generate actual traffic
(otherwise we would use the ping endpoint, which doesn't sleep 0.1s per request)


e.g.
```sh
# t1
kubectl get svc rng
# kubectl apply -f https://k8smastery.com/shpod.yaml
kubectl attach --namespace=shpod -ti shpod
ab -c 10 -n 1000 http://10.101.25.252/1

# t2
kubectl get events -w
# t3
kubectl get pods -w
```

### Discussion
* Above a given threshold, the liveness probe starts failing
  * (about 10 concurrent requests per backend should be plenty enough)
* When the liveness probe fails 3 times in a row, the container is restarted
* During the restart, there is less capacity available
* ...Meaning that the other backends are likely to timeout as well
* ...Eventually causing all backends to be restarted
* ...And each fresh backend gets restarted, too
* This goes on until load goes down, or we add capacity

This wouldn't be a good healthcheck in a real application!

### Better healthchecks
* We need to make sure that the healthcheck doesn't trip when performance degrades
due to external pressure
* Using a readiness check would have fewer effects
  * (but it would still be an imperfect solution)
* A possible combination:
  * readiness check with a short timeout / low failure threshold
  * liveness check with a longer timeout / higher failure threshold

### Healthchecks for redis
* A liveness probe is enough 
(it's not useful to remove a backend from rotation when it's the only one)
* We could use an exec probe running `redis-cli ping`

### Exec probes and zombies
* When using exec probes, we should make sure that we have a zombie reaper 
🤔🤔Wait,what?
* When a process terminates, its parent must call `wait()`/`waitpid()`
(this is how the parent process retrieves the child's exit status)
* In the meantime, the process is in zombie state
(the process state will show as `z` in `ps`, `top`...)
* When a process is killed, its children are orphaned and attached to PID 1
* PID 1 has the responsibility of reaping these processes when they terminate
* OK, but how does that affect us?

### PID 1 in containers
* On ordinary systems, PID 1(`/sbin/init`) has logic to reap porcesses
* In containers, PID 1 is typically our application process
(e.g. Apache, the JVM, NGINX, Redix...)
* These do not take care of reaping orphans
* If we use exec probes, we need to add a process reaper
* We can add tini to our images
* Or share the PID namespace between containers of a pod
  * (and have gcr.io/pause take care of the reaping)
* Discussion of this in Video - 10 Ways to Shoot Yourself in the Foot 
with Kubernetes, Will Surprise You

### Tini and redis ping in a liveness probe
1. Add `tini` to your own custom redis image
2. Change the kubercoins YAML to use your own image
3. Create a liveness probe in kuberoins YAML
4. Use `exec` handeler and run `tini -s -- redis-cli ping`
5. Example repo here: `github.com/BretFisher/redis-tini`

```yml
containers:
- name: redis
  image: custom-redis-image
  livenessProbe:
    exec:
      command:
      - /tini
      - -s
      - --
      - redis-cli
      - ping
    initialDelaySeconds: 30
    periodSeconds: 5
```

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our DockerCoin resources(for now):
```sh
kubectl delete -f https://k8smastery.com/dockercoins.yaml
```

### Managing configuration
* Some application need to be configured(obviously!)
* There are many ways for our code to pick up configuration:
  * command-line arguments
  * environment variables
  * configuration files
  * configuration servers(getting configuration from a database, an API...)
  * ... and more(because programmers can be very creative!)
* How can we do these things with containers and Kubernetes?

### Passing configuration to containers
* There are many ways to pass configuration to code running in a container:
  * baking it into a custom image
  * command-line arguments
  * environment variables
  * injecting configuration files
  * exposing it over the Kubernetes API
  * configuration servers
* Let's review these different strategies!

### Baking custom images
* Put the configuration in the image
(it can be in a configuration file, but also `ENV` or `CMD` actions)
* It's easy! It's simple!
* Unfortunately, it also has downsides:
  * multiplication of images
  * different images for dev, staging, prod ...
  * minor reconfigurations require a whole build/push/pull cycle
* `Avoid doing` it unless you don't have the time to figure out other options

### Command-line arguments
* Pass options to `args` array in the container specification
* Example(source):
```sh
args:
  - "--data-dir=/var/lib/etcd"
  - "--advertise-client-urls=http://127.0.0.1:2379"
  - "--listen-client-urls=http://127.0.0.1:2379"
  - "--listen-peer-urls=http://127.0.0.1:2380"
  - "--name=etcd"
```
* The options can be passed directly to the program that we run...
...or to a wrapper script that will use them to e.g. generate a config file

### Command-line arguments, pros の cons
* Works great when options are passed directly to the running program
(otherwise, a wrapper script can work around the issue)
* Works great when there aren't too many parameters
(to avoid a 20-lines `args` array)
* Requires documentation and/or understanding of the underlying program
("which parameters and flags do I need, again?")
* Well-suited for mandatory parameters(without default values)
* Not ideal when we need to pass a real configuration file anyway

### Environment variables
* Pass options through the `env` map in the container specification
* Example:
```yaml
env:
- name: ADMIN_PORT
  value: "8080"
- name: ADMIN_AUTH
  value: Basic
- name: ADMIN_CRED
  value: "admin:0pensesame!"
```
`value` must be a string! Make sure that numbers and fancy strings are quoted

🤔Why this weired `{name: xxx, value: yyy}` scheme?It will be revealed soon!

### The Downward API
* In the previous example, environment variables have fixed values
* We can also use a mechanism called the Downward API
* The Downward API allows exposing pod or container information
  * either through special files(we won't show that for now)
  * or through environment variables
* The value of these environment variables is computed when the container is started
* Remember: environment variables won't(can't) change after container start
* Let's see a few concrete examples!

### Exposing the pod's namespace
```yaml
- name: MY_POD_NAMESPACE
  valueFrom:
    fileRef:
      fieldPath: metadata.namespace
```
* Useful to generate FQDN of services
(in some contexts, a short name is not enough)
* For instance, the two commands should be equivalent:
```sh
curl api-backend
curl api-backend.$MY_POD_NAMESPACE.svc.cluster.local
```

### Exposing the pod's IP address

```yaml
- name: MY_POD_IP
  valueFrom:
    fieldRef:
      fieldPath: status.podIP
```
* Useful if we need to know our IP address
(we could also read it from `eth0`, but this is more solid)

### Exposing the container's resource limits
```yaml
- name: MY_MEM_LIMIT
  valueFrom:
    resouceFieldRef:
      containerName: test-container
      resource: limits.memory
```
* Useful for runtimes where memory is garbage collected
* Example: the JVM
(the memory available to the JVM should be set with the `-Xmx` flag)
* Best practice:set a memory limit, and pass it to the runtime
* Note: recent versions of the JVM can do this automatically
(see `JDK-8146115`) and `this blog post` for detailed examples

### More about the Downward API
* This documentation page tells more about these environment variables
* And `this one` explains the other way to use the Downward API
(through files that get created in the container filesystem)

### Environment variables, pros and cons
* Works great when the running program expects these variables
* Works great for optional parameters with reasonable defaults
(since the container image can provide these defaults)
* Sort of auto-documented
(we can see which environment variables are defined in the image, and their values)
* Can be (ab)used with longer values...
* ...You can put an entire Tomcat configuration file in an environment...
* ...But should you?
(Do it if you really need to, we're not judging! Bu we'll see better ways.)

### Injecting configuration files with ConfigMaps
* Sometimes, there is no way around it: we need to inject a full config file
* Kubernetes provides a mechanism for that purpose: `ConfigMaps`
* A ConfigMap is a Kubernetes resource that exits in a namespace
* Conceptually, it's a key/value map
(values are arbitrary strings)
* We can think about them in(at least) two different ways:
  * as holding entire configuration file(s)
  * as holding individual configuration parameters

Note: to hold sensitive information, we can use "Secrets", which are another typ of.. resource behaving very much like ConfigMaps.
We'll cover them just after!

### ConfigMaps storing entire files
* In this case, each key/value pair corresponds to a configuration file
* Key = name of the file
* Value = content of the file
* There can be one key/value pair, or as many as necessary
(for complex apps with multiple configuration files)
* Examples:
```sh
# Create a ConfigMap with a single key, "app.conf"
kubectl create configmap my-app-config --from-file=app.conf
# Create a ConfigMap with a single key, "app.conf" but another file
kubectl create configmap my-app-config --from-file=app.conf=app-prod.conf
# Create a ConfigMap with multiple keys (one per fiel in the config.d directory)
kubectl create configmap my-app-config --from-file=config.d/
```

### ConfigMaps storing individual parameters
* In this case, each key/value pair corresponds to a parameter
* Key = name of the parameter
* Value = value of the parameter
* Examples:
```sh
# Create a ConfigMap with two keys
kubectl create cm my-app-config \
  --from-literal=foreground=red \
  --from-literal=foreground=blue

# Create a ConfigMap from a file containing key=val pairs
kubectl create cm my-app-config \
  --from-env-file=app.conf
```

### Exposing ConfigMaps to containers
* ConfigMaps can be exposed as plain files in the filesystem of a container
  * this is achieved by declaring a volume and mounting it in the container
  * this is particularly effective for ConfigMaps containing whole files
* ConfigMaps can be exposed as environment variables in the container
  * this is achieved with the Downward API
  * this is particularly effective for ConfigMaps containing individual parameters
* Let's see how to do both!

### Passing a configuration file with a ConfigMap
* We will start a load balancer powered by HAProxy
* We will use the official haproxy image
* It expects to find its configuration in `/usr/local/etc/haproxy/haproxy.cfg`
* We will provide a simple HAProxy configuration
* It listens on port 80, and load balances connections between IBM and Google

### Creating the ConfigMap
Exercise
* Download our simple HAProxy config:
```sh
curl -O https://k8smastery.com/haproxy.cfg
```
* Create a ConfigMap named `haproxy` and holding the configuration file:
```sh
kubectl create configmap haproxy --from-file=haproxy.cfg
```
* Check what our ConfigMap looks like:
```sh
kubectl get configmap haproxy -o yaml
```

### Using the ConfigMap
* We are going to use the following pod definition:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: haproxy
spec:
  volumes:
  - name: config
    configMap:
      name: haproxy
  containers:
  - name: haproxy
    image: haproxy
    volumeMounts:
    - name: config
      mountPath: /usr/local/etc/haproxy/
```
### Using the ConfigMap
* Apply the resource definition from the previous slide

Exercise
* Create the HAProxy pod:
```sh
kubectl apply -f https://k8smastery.com/haproxy.yaml
```
* Check the IP address allocated to the pod, inside `shpod`:
```sh
kubectl attach --namespace=shpod -ti shpod
kubectl get pod haproxy -o wide
IP=$(kubectl get pod haproxy -o json | jq -r .status.podIP)
```

### Testing our load balancer
* The load balancer will send:
  * half of the connections to Google
  * the other half to IBM

Exercise
* Access the load balancer a few times:
```sh
curl $IP
curl $IP
curl $IP
```
We should see connections served by Google, and others served by IBM.
(Each server sends us a redirect page. Look at the URL that they send us to!)

### Exposing ConfigMaps with the Downward API
* We are going to run a Docker registry on a custom port
* By default, the registry listens on port 5000
* This can be changed by setting environment variable `REGISTRY_HTTP_ADDR`
* We are going to store the port number in a ConfigMap
* Then we will expose that ConfigMap as a container environment variable

### Creating the ConfigMap

Exercise
* Our ConfigMap will have a single key,`http.addr`:
```sh
kubectl create configmap registry --from-literal=http.addr=0.0.0.0:80
```
* Check our ConfigMap:
```sh
kubectl get configmap registry -o yaml
```

### Using the ConfigMap
* We are going to use the following pod definition:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: registry
spec:
  containers:
  - name: registry
    image: registry
    env:
    - name: REGISTRY_HTTP_ADDR
      valueFrom:
        configMapKeyRef:
          name: registry
          key: http.addr
```
* The resource definition from the previous slide:

Exercise
* Create the registry pod:
```sh
kubectl apply -f https://k8smastery.com/registry.yaml
```
* Check the IP address allocated to the pod
```sh
kubectl attach --namespace=shpod -ti shpod
kubectl get pod registry -o wide
IP=$(kubectl get pod registry -o json | jq -r .status.podIP)
```
* Confirm that the registry is available on port 80:
```sh
curl $IP/v2/_catalog
```

### Passwords, tokens, sensitive information
* For sensitive information, there is another special resource: Secrets
* Secrets and Configmaps work almost the same way
(we'll expose the differences on the next slide)
* The intent is different, though:
```
"You should use secrets for things which are actually
 secret like API keys, credentials, etc.and use config map
 for not-secret configuration data."
"In the future there will likely be some differentiators
 for secrets like rotation or support for backing the
 secret API w/HSMs, etc."
```
(Source: the author of both features)

### Differences between ConfigMaps and Secrets
* Secrets are base64-encoded when shown with `kubectl get secrets -o yaml`
  * keep in mind that this is just encoding, not encryption
  * it is very easy to automatically extract and decode secrets
* Secrets can be encrypted at rest
* With RBAC, we can authorize a user to access ConfigMaps, but not Secrets
(since they are two different kinds of resources)

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our pods:
```sh 
kubectl delete pod/haproxy pod/registry
```

### Exposing HTTP services with Ingress resources
* Services give us a way to access a pod or a set of pods
* Services can be exposed to the outside world:
  * with type `NodePort`(on a port > 30000)
  * with type `LoadBalancer`(allocating an external load balancer)
* What about HTTP services?
  * how can we expose `webui`, `rng`, `hasher`?
  * the Kubernetes dashboard?
  * all on the same IP and port?

### Exposing HTTP services
* If we use `NodePort` services, clients have to specify port numbers
(i.e. http://xxxxx:31234 instead of just http://xxxxx)
* `LoadBalancer` services are nice, but:
  * they are not available in all environments
  * they often carry an additional cost(e.g they provision an ELB)
  * they often work at OSI Layer 4(IP+Port) and not Layer 7(HTTP/S)
  * they require one extra step for DNS integration
  (waiting for the `LoadBalancer` to be provisioned;then adding it to DNS)
* We could build our own reverse proxy

### Building a custom reverse proxy
* There are many options available:
  Apache, HAProxy, Envoy Proxy, Gloo, NGINX, Traefik,...
* Most of these options require us to update/edit configuration files after each change
* Some of them can pick up virtual hosts and backends from a configuration store
* Wouldn't it be nice if this configuration could be managed with Kubernetes API?
* Enter Ingress resources!


### ingress vs. Ingress

ingress
* ingress definition: Going in, entering. The opposite of egress(leaving)
* In networking terms, ingress refers to handling incomming connections
* Could imply incoming to firewall, network, or in this case, a server cluter

Ingress
* Ingress(capital 1)in these slides means the Kubernetes Ingress resource
* Specific to HTTP/S

### Ingress resources
* Kubernetes API resource(`kubectl get ingress/ingresses/ing`)
* Designed to expose HTTP services
* Basic features:
  * load balancing
  * SSL termination
  * name-based virtual hosting
* Can also route to different services depending on:
  * URI path(e.g `/api` -> `api-service`, `static`->assets-service)
  * Client headers, including cookies(for A/B testing, canary deployment...)
  * and more

### Principle of operation
* Step 1:deploy an Ingress controller
  * Ingress controller = load balancing proxy + control loop
  * the control loop watches over Ingress resources, and configures the LB accordingly
  * these might be two separate processes(NGINX server + NGINX Ingress controller)
  * or a single app that knows how to speak to Kubernetes API(Traefik)
* Step 2: set up DNS(usually)
  * associate external DNS entries with the load balancer or host address
* Step 3: create Ingress resources for our Services resources
  * these resources contain rules for handling HTTP/S connections
  * the Ingress controller picks up these resources and configures the LB
  * connections to the Ingress LB will be processed by the rules


### Ingress in action: NGINX
* We will deploy the NGINX Ingress controller first
  * this is a popular, yet arbitrary choice, the `docs` list over a dozen options
* For DNS, we will use `nip.io`
  * `*.127.0.0.1.nip.io` resolves to `127.0.0.1`
  * we do this so we can use various FQDN's without editing our `hosts` file
* We will create Ingress resources for various HTTP-based Services

### Deploying pods listening on port 80
* We want our Ingress load balancer to be available on port 80
* We could do that with a `LoadBalancer` service
  * ...but it requires support from the underlying infrastructure
  * minikube and MicroK8s don't work with it
  * ...but Docker Desktop supports it for `localhost`!
* We could use pods specifying `hostPort: 80`
  * ...but with most CNI plugins, this `doesn't work or requires additional setup`
* We could use a `NodePort` service
  * ...but that requires `changing the --service-node-port-range` flag in the API service
* Last resort: the `hostNetwork` mode

### Without `hostNetwork`
* Normally, each pod gets its own network namespace
(sometimes called sandbox or network sandbox)
* An IP address is assigned to the pod
* This IP address is routed/connected to the cluster network
* All containers of that pod are sharing that network namespace
(and therefore using the same IP address)


### With `hostNetwork: true`
* No network namespace gets created
* The pod is using the network namespace of the host
* It "sees" (and can use) the interfaces(and IP addresses) of the host(VM on macOS/Win)
* The pod can receive outside traffic directly, on any port
* Downside: with most network plugins, network polices won't work for that pod
  * most network policies work at the IP address level
  * filtering that pod = filtering traffic from the node

### What you will use now
* Docker Desktop
  * no built-in ingress installer, we'll provide you YAML
  * Ignores `hostNetwork`, but Service `type: LoadBalancer` works with `localhost`!
* minikube
  * has a built-in NGINX installer `minikube addons enable ingress`
  * But, let's use YAML we provide for learning purposes
  * `hostNetwork: true` enabled on pod works for minikube IP
* MicroK8s:
  * has a built-in NGINX installer `microk8s.enable ingress`
  * let's use YAML we provide anyway for learning purposes
  * `hostNetwork: true` enabled on pod works for MicroK8s host IP


### Running NGINX on our cluster
* Now let's deploy the NGINX controller. Pick your distro:

Exercise
* Apply the YAML
```sh
# for Docker Desktop, create Service with LoadBalancer
kubectl apply -f https://k8smastery.com/ic-nginx-lb.yaml
# for minikube/MicroK8s, create Service with hostNetwork
kubectl apply -f https:/k8smastery.com/ic-nginx-hn.yaml
```
* Check the pod Status
```sh
kubectl describe -n ingress-nginx deploy/nginx-ingress-controller
kubectl get all -n ingress-nginx
```

### Deploying the NGINX Ingress controller
* We need the YAML templates from the `kubernetes/ingress-nginx` project

The two main sections in the YAML are:
* NGINX Deployment(or DaemonSet) and all its required resources
  * Namespace
  * ConfigMaps(storing NGINX configs)
  * ServicesAccount(authenticate to Kubernetes API)
  * Role/ClusterRole/RoleBindings(authorization to API parts)
  * LimitRange(limit cpu/memory of NGINX)
* `Service to` expose NGINX on 80/443
  * different for each Kubernetes distribution

```sh
kubectl apply -f https://k8smastery.com/ic-nginx-lb.yaml
kubectl describe -n ingress-nginx deploy/nginx-ingress-controller
kubectl get all -n ingress-nginx
```

### Checking that NGINX
* If NGINX started correctly, we now have a web server listening on each node

Exercise
* Direct your browser to your Kubernetes IP on port 80

We should get a `404 page not found` error.

This is normal: we haven't provided any Ingress rule yet.

### Setting up DNS

* To make our lives easier, we will use `nip.io`
* Check out `http://cheddar.A.B.C.D.nip.io`
(replacing A.B.C.D with the IP address of your Kubernetes IP)
* We should get the same `404 page not found` error
(meaning that our DNS is "set up properly" so to speak!)

```sh
cheddar.127.0.0.1.nip.io
```

### Setting up host-based routing ingress rules
* We are going to use `errm/cheese` images
(there are `3 tags available`: wensleydale, cheddar, stilton)
* These images contain a simple static HTTP server sending a picture of cheese
* We will run 3 deployments(one for each cheese)
* We will create 3 services(one for each deployment)
* Then we will create 3 ingress rules(one for each service)
* We will route `<name-of-cheese>.A.B.C.D.nip.io` to the corresponding deployment


### Running cheesy web servers

Exercise
* Run all three deployments:
```sh
kubectl create deployment cheddar --image=errm/cheese:cheddar
kubectl create deployment stilton --image=errm/cheese:stilton
kubectl create deployment wensleydale --image=errm/cheese:wensleydale
```
* Create a service for each of them:
```sh
kubectl expose deployment cheddar --port=80
kubectl expose deployment stilton --port=80
kubectl expose deployment wensleydale --port=80
```

### What does an ingress resource look like?

Here is a minimal host-based ingress resource:
```sh
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: cheddar
spec:
  rules:
  - host: cheddar.A.B.C.D.nip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: cheddar
          servicePort: 80
```
(In 1.14 ingress moved from the extensions API to networking)

### Creating our first ingress resources

Exercise
* Download our YAML `curl -O https://k8smastery.com/ingress.yaml`
* Edit the file `ingress.yaml` which has three Ingress resources
* Replace the A.B.C.D with your Kubernetes IP (`127.0.0.1` for `localhost`)
* Apply the file `kubectl apply -f ingress.yaml`
* Open `http://cheddar.A.B.C.D.nip.io`
(An Image of a piece of cheese should show up.)

### Bring up the other Ingress resources
Exercise
* Open `http://stilton.A.B.C.D.nip.io`
* Open `http://wensleydale.A.B.C.D.nip.io`

* Different cheeses should show up for each URL

```
http://cheddar.127.0.0.1.nip.io
http://stilton.127.0.0.1.nip.io
http://wensleydale.127.0.0.1.nip.io
```

### Adding features to a Ingress resource
* Reverse proxies have lots of features
* Let's add a 301 redirect to a new Ingress resource using annotations
* It will apply when any other path is used in URL that we didn't already add

Exercise
* Create a redirect
```sh
kubectl apply -f https://k8smastery.com/redirect.yaml
```
* Open http://<anything>.A.B.C.D.nip.io or localhost or A.B.C.D
* It should immediately redirect to google.com

```
http://somethingelse.127.0.0.1.nip.io
```


### View Ingress resources

* Let's inspect some ingress resources

Exercise
* List all Ingress resources in the `default` namespace
```sh
kubectl get ingress
```
* Get the details on the `cheddar` Ingress resource
```sh
kubectl describe ingress cheddar
```
* Get the details on the `my-google` Ingress resource
```sh
kubectl describe ingress my-google
```
* Output `stilton` in YAML
```sh
kubectl get ingress/stilton -o yaml
```

### Swapping NGINX for Traefik
* Traefik is a proxy with built-in Kubernetes Ingress support
* It has a web dashboard, built-in Let's Encrypt, full TCP support, and more
* Most importantly: Traefik releases are named after cheeses
* The `Traefik documentaion` tells us to pick between Deployment and DaemonSet
* We are going to use a DaemonSet so that each node can accept connections
* We provide a YAML file which is essentially the sum of:
  * `Traefik's DaemonSet resources`(patched with `hostNetwork` and tolerations)
  * `Traefik's RBAC rules` allowing it to watch necessary API objects
* We will make a minor change to the `YAML provided by Traefik` to enable `hostNetwork` for MicroK8s/minikube
* For Docker Desktop we'll add a `type: LoadBalancer` to the Service

### Removing NGINX from our cluster
* Before starting Traefik, let's remove the NGINX controller
* This won't remove Services or Ingress resources
* But it will make them unavailable from outside the cluster

Exercise
* Delete our NGINX controller and related resources:
```sh
# for Docker Desktop, create Service with LoadBalancer
kubectl delete -f https://k8smastery.com/ic-nginx-lb.yaml

# for minikube/MicroK8s, create Service with hostNetwork
kubectl delete -f https:/k8smastery.com/ic-nginx-hn.yaml
```
* Also remove the redirect Ingress resource. It only worked in NGINX
```sh
kubectl delete -f https:/k8smastery.com/redirect.yaml
```

### Running Traefik on our cluster
* Now let's deploy the Traefik Ingress controller

Exercise
* Apply the YAML
```sh
# for Docker Desktop with LoadBalancer
kubectl apply -f https://k8smastery.com/ic-traefik-lb.yaml
# kubectl delete -f https://k8smastery.com/ic-traefik-lb.yaml
# for minikube/MicroK8s hostNetwork
kubectl apply -f https:/k8smastery.com/ic-traefik-hn.yaml
```
* Check the pod Status
```sh
kubectl describe -n kube-system ds/traefik-ingress-controller
kubectl get all -n kube-system
```

### Checking that Traefik runs correctly
* If Traefik started correctly, we can refresh a cheese and it still works

Exercise
* Refresh `http://cheddar.A.B.C.D.nip.io`

```
http://cheddar.127.0.0.1.nip.io
http://stilton.127.0.0.1.nip.io
http://wensleydale.127.0.0.1.nip.io
```

### Traefik web UI
* Traefik provides a web dashboard on container port 8080
* For those using the `LoadBalancer` method(Docker Desktop), it's enabled

Exercise
* if using Docker Desktop, go to `http://localhost:8080`

* For those use `hostNetwork`, this could be a problem
* The container won't start if anything is listening on `<host IP>:8080`
* On MicroK8s, Kubernetes API runs on 8080 😂
* For those using minikube, you can un-comment the YAML and re-apply
* You could also edit the resource(s) and manually add the details, e.g.
  * `kubectl edit -n kube-system ds/traefik-ingress-controller`


### What about Traefik 2.x IngressRoute resources?
* We've been using Traefik 2.x as the Ingress controller
* Traefik released 2.0 in late 2019
* Their documentation talks about IngressRoute resource
* But IngressRoute is not a build-in resource of Kubernetes
* Traefik 2.x now supports a custom CRD(Custom Resource Definition)
* We'll explore why in a bit


### Using multiple ingress controllers
* You can have multiple ingress controllers active simultaneously
(e.g. Traefik, Gloo, and NGINX)
* You can even have multiple instances of the same controller
(e.g. one for internal, another for external traffic)
* The `kubernetes.io/ingress.class` annotation can be used to tell which one to use
* It's OK if multiple ingress controllers configure the same resource
(it just means that the service will be accessible through multiple paths)

### Ingress resources: the good
* The traffic flows directly from the ingress load balancer to the backends
  * it doesn't need to go through the `ClusterIP`
  * in fact, we don't even need a `ClusterIP`(we can use a headless service)
* The load balancer can be outside of Kubernetes
(as long as it has access to the cluster subnet)
* This allows the use of external(hardware, physical machines...) load balancers
* Annotations can encode special features
(rate-limiting, A/B testing, session stickiness, etc.)

### Ingress resources: the bad(cough Annotations cough)
* Aforementioned "special features" are not standardized yet
* Some controller will support them; some won't
* Even relatively common features(stripping a path prefix) can differ:
  * `traefik.ingress.kubernetes.io/rule-type: PathPrefixStrip`
  * `Ingress.kubernetes.io/rewrite-target: /`
* This should eventually stabilize
(remember that ingresses are currently `apiVersion: networking.k8s.io/v1beta1`)
* Annotations are not validated in CLI
* Some proxies provide a CRD(Custom Resource Definition) option

### When not to use built-in Ingress resources
* You need features beyond Ingress including:
  * TCP support, traffic splitting, mTLS, egress, service mesh
  * response transformation, routing to 2+ services
* You have external load balancers(like AWS ELBs) which route to NodePorts
* You don't need externally available HTTP services on the default ports


### Using CRD's as alternatives to Ingress resources
* Due to the limits of the built-in ingress, many projects are moving to CRD's
* For example, Traefik 2.x has a IngressRoute CRD option
* Ambassador, a controller for Envoy proxy, uses a Mapping CRD
* These CRD proxy options do ingress plus more (sometimes called API Gateways):
  * TCP Support(anything beyond HTTP/HTTPS)
  * Traffic splitting, rate limiting, circuit breaking, etc
  * Complex traffic routing, request and response transformation
* Once we consider CRD's, many more proxy options are available:
  * Envoy Proxy based (Gloo, Ambassador, Contour)
  * Other Proxies(Tyk, Traefik, Kong, KrakenD)
* Eventually, some more advanced features might be added to "Ingress Resources"
* We'll cover more after we learn about CRD's and Operators

### Cleanup
Let's cleanup before we start the next lecture!

Exercise
* remove our ingress controller:
```sh
# for Docker Desktop with LoadBalancer
kubectl delete -f https://k8smastery.com/ic-traefik-lb.yaml
# for minikube/MicroK8s with hostNetwork
kubectl delete -f https:/k8smastery.com/ic-traefik-hn.yaml
```
* remove our ingress resources:
```sh
kubectl delete -f ingress.yaml
kubectl delete -f https:/k8smastery.com/redirect.yaml
```
* remove our cheeses
```sh
kubectl delete svc/cheddar svc/stilton svc/wensleydale
kubectl delete deploy/cheddar deploy/stilton deploy/wensleydale
```
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------

------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------


### Image vs. Container
* An Image is the application we want to run
* A Container is an instance of that image running as a process
* You can have many containers running off the same image
* In this lecture our image will be the Nginx web server
* Docker's default image "registry" is called Docker Hub
  * (hub.docker.com)

```sh
docker container run --publish 80:80 nginx
# ctrl + c
# docker container run --publish 8080:80 nginx
# docker container run --publish 8080:80 --detach nginx
# docker container ls
# docker container stop 690
# docker container ls -a
```

### docker container run --publish 80:80 nginx
1. Downloaded image 'nginx' from Docker Hub
2. Started a new container from that image
3. Opened port 80 on the host IP
4. Routes that traffic to the container IP, port 80
```
Note you'll get a "bind" error if the left number [host port]
is being used by anthing else, even another container.
You can use any port you want on the left, like 8080:80
or 8888:80,then use localhost:8888 when testing
```

```sh
docker container run --publish 80:80 --detach --name webhost nginx
docker container ls -a
docker container logs webhost
docker container top webhost
docker container --help
```

### What happens in 'docker container run'
1. Looks for that image locally in image cache, doesn't find anything
2. Then looks in remote image repository(defaults to Docker Hub)
3. Downloads the latest version(nginx:latest by default)
4. Creates new container based on that image and prepares to start
5. Give it a virtual IP on a private network inside docker engine
6. Opens up port 80 on host and forwards to port 80 in container
7. Starts container by using the CMD in the image Dockerfile

### Example Of Changing The Default
```sh
docker container run --publish 8080:80 --name webhost -d nginx:1.11 nginx -T
# 8080 -> change host listening port
# 1.11 -> change version of image
# nginx -T -> change CMD run on start
```

### Containers aren't Mini-VM's
* They are just processes
* Limited to what resources they can access
* Exit when process stops

```sh
docker run --name mongo -d mongo
docker ps
docker top mongo
ps aux
docker stop mongo
ps aux | grep mongo
docker start mongo
docker ps
docker top mongo
```

### Assignment: Manage Multiple Containers
* docs.docker.com and `--help` are your friend
* Run a `nginx`, a `mysql`, and a `httpd`(apache) server
* Run all of the `--detach`(or `-d`), name them with `--name`
* nginx should listen on `80:80`, httpd on `8080:80`, mysql on `3306:3306`
* When running `mysql`, use the `--env` option(or `-e`) to pass in 
`MYSQL_RANDOM_ROOT_PASSWORD=yes`
* Use `docker container logs` on mysql to find the random password it created on startup
* Clean it all up with `docker container stop` and `docker container rm`
(both can accept multiple names or ID's)
* Use `docker container ls` to ensure everything is correct before and after cleanup

### Assignment Reminder
* This lecture is ANSWERS to previous assignment
* Try the previous lecture (Assignment details) first on your own before
watching this video
* Forcing yourself to figure this assignment out by sudying commands,
docs.docker.com, etc. will help this stuff stick in your brain better
then just watching me do it! 

```sh
docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
docker container logs db
# find & copy-> GENERATED ROOT PASSWORD: Aekia1fiighaengeethe0eihai2ailij

docker container run -d --name webserver -p 8080:80 httpd
docker ps

docker container run -d --name proxy -p 80:80 nginx
docker container ls

curl localhost
curl localhost:8080
 
docker container stop nginx httpd mysql
docker container stop # tap completion

docker container ls -a
docker container rm # tap completion again

docker container rm 4f444c35ebf7 8f9ccb6b0553 0827cf019e82 bb7958b14a40

docker ps -a
docker image ls
```

### What's Going On In Containers
* docker container top `- process list in one container`
* docker container inspect `- details of one container config`
* docker container stats `- performance stats for all containers`

```sh
docker container run -d --name nginx nginx
docker container run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql
docker container ls
docker container top mysql
docker container top nginx
docker container inspect mysql

docker container stats --help
docker container stats
```

### Getting a Shell Inside Containers
* `docker container run -it` - start new container interactively
* `docker container exec -it` - run additional command in existing container
* Different Linux distros in containers

```sh
docker container run -it --name proxy nginx bash
exit
docker container ls -a

docker container run -it --name ubuntu ubuntu
apt-get update
apt install -y curl
curl baidu.com
exit

docker container ls -a

docker container start --help
docker container start -ai ubuntu
curl google.com
exit

docker container exec --help
docker container exec -it mysql bash
ps aux
exit
docker container ls

docker pull alpine
docker container run -it alpine sh
apk
```

### Docker Networks: Concepts
* Review of `docker container run -p`
* For local dev/testing, networks usually "just work"
* Quick port check with `docker container port <container>`
* Learn concepts of Docker Networking
* Understand how network packets move around Docker

### Docker Networks Defaults
* Each container connected to a private virtual network "bridge"
* Each virtual network routes through NAT firewall on host IP
* All containers on a virtual network can talk to each other without -p
* Best practice is to create a new virtual network for each app:
  * network "my_web_app" for mysql and php/apache containers
  * network "my_api" for mongo and nodejs containers

### Docker Networks Cont.
* "Batteries Included, But Removable"
  * Defaults work well in many cases, but easy to swap out parts to customize it
* Make new virtual networks
* Attach containers to more then one virual network(or none)
* Skip virual networks and use host IP(--net=host)
* Use different Docker network drivers to gain new abilities
* and much more...

```sh
docker container run -p 80:80 --name webhost -d nginx
docker container port webhost
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' webhost
ifconfig en0
```

### Docker Networks: CLI Management
* Show networks `docker network ls`
* Inspect a network `docker network inspect`
* Create a network `docker network create --driver`
* Attach a network to container `docker network connect`
* Detach a network from container `docker network disconnect`

```sh
docker network ls
docker network inspect bridge

docker network create my_app_net
docker network ls

docker network create --help
docker container run -d --name new_nginx --network my_app_net nginx
docker network inspect my_app_net
docker network connect 21c7d8642536 b3f503669e05
docker container inspect b3f503669e05

docker network disconnect 21c7d8642536 b3f503669e05
docker container inspect b3f503669e05
```

### Docker Networks: Default Security
* Create your apps so frontend/backend sit on same Docker network
* Their inter-communication never leaves host
* All externally exposed ports closed by default
* You must manually expose via `-p`, which is better default security!
* This gets even better later with Swarm and Overlay networks

### Docker Networks: DNS
* Understand how DNS is the key to easy inter-container comms
* See how it works by default with custom networks
* Learn how to use `--link` to enable DNS on default bridge network
```sh
docker container ls
docker network inspect my_app_net

docker container run -d --name my_nginx --network my_app_net nginx
docker network inspect my_app_net

docker container exec -it my_nginx ping new_nginx
```

### Docker Networks: DNS
* Containers shouldn't rely on IP's for inter-communication
* DNS for friendly names is built-in if you use custom networks
* You're using custom networks right?
* This gets way easier with Docker Compose in future Section

### Assignment Requirements: CLI App Testing
* Know how to use `-it` to get shell in container
* Understand basics of what a Linux distributions is like Ubuntu and CentOS
* Know how to run a container🐻

### Assignment: CLI App Testing
* Use different Linux distro containers to check `curl` cli tool version
* Use two different terminal windows to start bash in both `centos:7` and
 `ubuntu:14.04`, using `-it`
* Learn the `docker container --rm` option so you can save cleanup
* Ensoure `curl` is installed and on latest version for that distro
  * ubuntu: `apt-get update && apt-get install curl`
  * centos: `yum update curl`
* Check `curl --version`

```sh
#t1
docker container run --rm -it centos:7 bash
yum update curl -y 

#t2
docker ps -a
docker container run --rm -it ubuntu:14.04 bash
apt-get update && apt-get install -y curl
curl --version
```

### Assignment Requirements: DNS RR Test
* Know how to use -it go get shell in container
* Understand basics of what a Linux distribution is like Ubuntu and CentOS
* Know how to run a container 
* Understand basics of DNS records

### Assignment: DNS Round Robin Test
* Ever since Docker Engine 1.11, we can have multiple containers on a created network respond to the same DNS address
* Create a new virtual network(default bridge driver)
* Create two containers from `elasticsearch:2` image
* Research and use `-network-alias search` when creating them to give them an additional DNS name to respond to
* Run `alpine nslookup search` with `--net` to see the two
containers list for the same DNS name
* Run `centos curl -s search:9200` with `--net` multiple times until you see both "name" fields show
```sh
docker network create dude
docker container run -d --net dude --net-alias search elasticsearch:2
# --net-alias and --network-alias both work
docker container run --rm --net dude alpine nslookup search
docker container run --rm --net dude centos curl -s search:9200
docker container run --rm --net dude centos curl -s search:9200
docker container run --rm --net dude centos curl -s search:9200
docker container run --rm --net dude centos curl -s search:9200
### Test DNS Round Robin
docker container ls
docker container rm -f kind_darwin fervent_jackson
```

### This Section:
* All about images, the building blocks of containers
* What's in an image (and what isn't)
* Using Docker Hub registry
* Managing our local image cache
* Building our own images

### What's In An Image (And What Isn't)
* App binaries and dependencies 
* Metadata about the image data and how to run the image
* Official definition: "An Image is an ordered collection of root
  filesystem changes and the corresponding execution parameters for use within a container runtime"
* Not a complete OS. No kernel, kernel modules(e.g. drivers)
* Small as one file(your app binary) like a golang static binary
* Big as a Ubuntu distro with apt, and Apache, PHP, and more installed

### This Lecture:
* Basics of Docker Hub(hub.docker.com)
* Find Official and other good public images
* Download images and basics of image tags

### This Lecture: Review
* Docker Hub, "the apt package system for Containers"
* Official images and how to use them
* How to discern "good" public images
* Using different base images like Debian or Alpine

### This Lecture:
* Image layers
* union file system
* `history` and `inspect` commands
* copy on write

```sh
docker image ls
docker history nginx:latest
# Oops that's old command format
docker history mysql

docker image inspect
# docker inspect (old way)
# returns JSON metadata about the image
docker image inspect nginx
```

### Image and Their Layers: Review
* Images are made up of file system changes and metadata
* Each layer is uniquely identified and only stored once on a host
* This saves storage space on host and transfer time on push/pull
* A container is just a single read/write layer on top of image
* `docker image history` and `inspect` commands can teach us

### This Lecture: Requirements
* Know what container and images are
* Understand image layer basics
* Understand Docker Hub basics

### This Lecture:
* All about image tags
* How to upload to Docker Hub
* Image ID vs. Tag

```sh
docker image tag --help
docker image ls

docker pull mysql/mysql-server
docker pull nginx:mainline
docker image tag nginx lotteryjs/nginx
docker image tag --help
docker image ls
docker login
cat ~/.docker/config.json
docker image push lotteryjs/nginx
docker image tag lotteryjs/nginx lotteryjs/nginx:testing
docker image ls
docker image push lotteryjs/nginx:testing
```

### This Lecture: Review
* Properly tagging images
* Tagging images for upload to Docker Hub
* How tagging is related to image ID
* The Latest Tag
* Logging into Docker Hub from docker cli
* How to create private Docker Hub images

e.g. docker build -f some-dockerfile

```sh
cd dockerfile-sample-1
docker image build -t customnginx .
```

```sh
cd dockerfile-sample-2
ll
vim Dockerfile
docker container run -p 80:80 --rm nginx
docker image build -t nginx-with-html .
docker container run -p 80:80 --rm nginx-with-html
docker image ls
docker image tag --help
docker image tag nginx-with-html:latest lotteryjs/nginx-with-html:latest
docker image ls
```

### Assignment: Build Your Own Image
* Dockerfiles are part process workflow and part art
* Take existing Node.js app and Dockerize it
* Make `Dockerfile`. Build it. Test it. Push it. (rm it). Run it.
* Expect this to be iterative. Rarely do I get it right the first time.
* Details in `dockerfile-assignment-1/Dockerfile`
* Use the Alpine version of the official 'node' 6.x image
* Expected result is web site at http://localhost
* Tag and push to your Docker Hub account(free)
* Remove your image from local cache, run again from Hub

```sh
cd dockerfile-assignment-1
ll
vim Dockerfile
```

```yaml
# Instructions from the app developer
# - you should use the 'node' official image, with the alpine 6.x branch
FROM node:6-alpine
# - this app listens on port 3000, but the container should launch on port 80
  #  so it will respond to http://localhost:80 on your computer
EXPOSE 3000
# - then it should use alpine package manager to install tini: 'apk add --update tini'
RUN apk add --update tini
# - then it should create directory /usr/src/app for app files with 'mkdir -p /usr/src/app'
RUN mkdir -p /usr/src/app
# - Node uses a "package manager", so it needs to copy in package.json file
WORKDIR /usr/src/app
COPY package.json package.json
# - then it needs to run 'npm install' to install dependencies from that file
RUN npm install && npm cache clean
# - to keep it clean and small, run 'npm cache clean --force' after above
# - then it needs to copy in all files from current directory
COPY . .
# - then it needs to start container with command '/sbin/tini -- node ./bin/www'
CMD [ "tini", "--", "node", "./bin/www" ]
# - in the end you should be using FROM, RUN, WORKDIR, COPY, EXPOSE, and CMD commands
```

```sh
docker build -t testnode .
docker container run  --rm -p 80:3000 testnode
docker images
docker tag --help
docker tag testnode lotteryjs/testing-node
docker push --help
docker push lotteryjs/testing-node
docker image rm lotteryjs/testing-node
docker container run --rm -p 80:3000 lotteryjs/testing-node
```

### Section Overview
* Defining the problem of persistent data
* Key concepts with containers: immutable, ephemeral
* Learning and using Data Volumes
* Learning and using Bind Mounts
* Assignments

### Container Lifetime & Persistent Data
* Containers are usually immutable and ephemeral
* "immutable infrastructure":only re-deploy containers, never change
* This is the ideal scenario, but what about databases, or unique data?
* Docker gives us features to ensure these "separation fo concerns"
* This is known as "persistent data"
* Two ways: Volumes and Bind Mounts
* Volumes: make special location outside of container UFS
* Bind Mounts: link container path to host path


### Persistent Data: Volumes
* VOLUME command in Dockerfile

```sh
docker pull mysql
# Note you might want to do a `docker volume prune` to
# cleanup unused volumes and make it easier to see what
# you're doing here
docker image inspect mysql
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql
docker container inspect mysql
docker volume inspect bd2dd9f5ecabc7d9bd6779436b8fdbee1426613601e24ef7d94ef77b497b7c1c
docker container run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql
docker volume ls
docker container stop mysql
docker container stop mysql2
docker container ls
docker container ls -a
docker volume ls
docker container rm mysql mysql2
docker volume ls
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql  mysql
docker volume ls
docker volume inspect mysql-db
docker container rm -f mysql
docker container run -d --name mysql3 -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql  mysql
docker volume ls
docker container inspect mysql3
```

docker volume create
* required to do this before "docker run" to use custom drivers and labels

docker volume create --help

### Persistent Data: Bind Mounting
* Maps a host file or directory to a container file or directory
* Basically just two locations pointing to the same file(s)
* Again, skips UFS, and host files overwrite any in container
* Can't use in Dockerfile, must be at `container run`
* `... run -v /Users/bret/stuff:/path/container` (mac/linux)
* `... run -v //c/Users/bret/stuff:/path/container` (windows)

```sh
cat dockerfile-sample-2
ll
pcat Dockerfile
docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx
docker container run -d --name nginx2 -p 8080:80 nginx
ll
# t2
docker container exec -it nginx bash
cd /usr/share/nginx/html
ls -al
# t1
touch testme.txt
# t2
ls -al
# t1
echo "is it me you're looking for" > testme.txt
```

### Assignment: Named Volumes
* Database upgrade with containers
* Create a `postgres` container with named volume psql-data using version `9.6.1`
* Use Docker Hub to learn `VOLUME` path and versions needed to run it
* Check logs, stop container
* Create a new `postgre` container with same named volume using `9.6.2`
* Check logs to validate
* (this only works with patch versions, most SQL DB's require manual commands
to upgrade DB's to major/minor versions,i.e. it's DB limitation not a container one)
```sh
docker container run -d --name psql -v psql:/var/lib/postgresql/data postgres:9.6.1
docker container logs -f psql
docker container stop psql
docker container run -d --name psql2 -v psql:/var/lib/postgresql/data postgres:9.6.2
docker container ps -a
docker volume ls
docker container logs psql2
```

### Assignment: Bind Mounts
* Use a Jekyll "Static Site Generator" to start a local web server
* Don't have to be web developer: this is example of bridging the gap
between local file access and apps running in containers
* source code is in the course repo under `bindmount-sample-1`
* We edit files with editor on our host using native tools
* Container detects changes with host files and updates web server
* start container with `docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve`
* Refresh our browser to see changes
* Change the file in `_posts\` and refresh browser to see changes

```sh
cd bindmount-sample-1
ls -la
docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
```

### Docker Compose
* Why: configure relationships between containers
* Why: save our docker container run settings in easy-to-read file
* Why: create one-liner developer environment startups
* Comprised of 2 separate but related things
* 1. YAML-formatted file that describes our solution options for:
  * containers
  * networks
  * volumes
* 2. A CLI tool `docker-compose` used for local dev/test automation with those YAML files

### docker-compose.yml
* Compose YAML format has it's own versions: 1, 2, 2.1, 3, 3.1
* YAML file can be used with `docker-compose` command for local docker automation or...
* With `docker` directly in production with Swarm(as of v1.13)
* `docker-compose --help`
* `docker-compose.yml` is default filename, but any can be used with
`docker-compose -f`

```sh
cd compose-sample-2
```

### docker-compose CLI
* CLI tools comes with Docker for Windows/Mac, but separate download for Linux
* Not a production-grade tool but ideal for local development and test
* Two most common commands are
  * docker-compose up # setup volumes/networks and start all containers
  * docker-compose down # stop all containers and remove con/vol/net
* If all your projects had a `Dockerfile` and `docker-compose.yml`
then "new developer onboarding" would be:
  * git clone github.com/some/software
  * docker-compose up
```sh
cd compose-sample-2
ls -la
cat docker-compose.yml
docker-compose up
# browser -> http://localhost
# ctrl + c
docker-compose up -d
docker-compose logs
docker-compose --help
docker-compose ps
docker-compose top
docker-compose down
```

### Assignment: Writing A Compose File
* Build a basic compose file for a Drupal content management system website.
Docker Hub is your friend
* Use the `drupal` image along with the `postgres` image
* Use `ports` to expose Drupal on 8080 so you can `localhost:8080`
* Be sure to set `POSTGRES_PASSWORD` for postgres
* Walk though Drupal setup via browser
* Tip: Drupal assumes DB is `localhost`, but it's service name
* Extra Credit: Use volumes to store Drupal unique data

```sh
docker pull drupal
docker image inspect drupal
```

```yaml
version: '2'

services:
  drupal:
    image: drupal:8.2
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles       
      - drupal-sites:/var/www/html/sites      
      - drupal-themes:/var/www/html/themes
  postgres:
    image: postgres:9.6
    environment:
      - POSTGRES_PASSWORD=mypasswd

volumes:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
```


```sh
docker-compose down -v
# Remove named volumes declared in the `volumes`
# section of the Compose file and anonymous volumes
# attached to containers.
```

### Using Compose to Build
* Compose can also build your custom images
* Will build them with `docker-compose up` if not found in cache
* Also rebuild with `docker-compose build`
* Great for complex builds that have lots of vars or build args

```sh
cd compose-sample-3/
docker-compose up
# localhost
docker-compose down
docker image ls
docker-compose down --help
docker image rm nginx-custom
docker image ls
# remove -> image: nginx-custom
docker-compose up -d
docker image ls
docker-compose down --rmi local
```

### Assignment: Build and Run Compose
* "Building custom `drupal` image for local testing"
* Compose isn't just for developers. Testing apps si easy/fun!
* Maybe your learning Drupal admin, or are a software tester
* Start with Compose file from previous assignment
* Make your `Dockerfile` and `docker-compose.yml` in dir `compose-assignment-2`
* Use the `drupal` image along with the `postgres` image as before
* Use `README.md` in that dir for details

```sh
cd compose-assignment-2
```

Dockerfile
```dockerfile
FROM drupal:8.6


RUN apt-get update && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html/themes

RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git \
    && chown -R www-data:www-data bootstrap

WORKDIR /var/www/html
```

docker-compose.yml

```yaml
version: '2'
# NOTE: move this answer file up a directory so it'll work

services:

  drupal:
    container_name: drupal
    image: custom-drupal
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles       
      - drupal-sites:/var/www/html/sites      
      - drupal-themes:/var/www/html/themes
 
  postgres:
    container_name: postgres
    image: postgres:9.6
    environment:
      - POSTGRES_PASSWORD=mypasswd
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

```

### Your First Swarm Service

[Docker Swarm Docs](https://docs.docker.com/swarm/)

[Play with Docker](https://labs.play-with-docker.com/)
* Templates
  * 3 Managers and 2 Workers
```sh
docker service create --name hello --replicas 3 --detach=false --publish 8080:80 nginx
```

### Containers Everywhere = New Problems
* How do we automate container lifecycle?
* How can we easily scale out/in/up/down?
* How can we ensure our containers are re-created if they fail?
* How can we replace containers are-created if they fail?
* How can we control/track where containers get started?
* How can we create cross-node virtual networks?
* How can we ensure only trusted servers run our containers?
* How can we store secrets, keys, passwords and get them to the right container(and only that container)?

### Swarm Mode:Built-In Orchestration
* Swarm Mode is a clustering solution built inside Docker
* Not related to Swarm "classic" for pre-1.12 versions
* Added in 1.12(Summer 2016)via SwarmKit toolkit
* Enhanced in 1.13(January 2017) via Stacks and Secrets
* Not enabled by default, new commands once enabled
  * docker swarm
  * docker node
  * docker service
  * docker stack
  * docker secret

### docker swarm init: What Just Happened?
* Lots of PKI and security automation
  * Root Signing Certificate created  for our Swarm
  * Certificate is issued for first Manager node
  * Join tokens are created
* Raft database created to store root CA, configs and secrets
  * Encrypted by default on disk(1.13+)
  * No need for another key/value system to hold orchestration/secrets
  * Replicates logs amongst Managers via mutual TLS in "control plane" 

### Create Your First Service and Scale It Locally

```sh
docker swarm init
docker node ls
# ...MANAGER STATUS
# ...Leader
docker node help
docker swarm help
docker service help

docker service create alpine ping 8.8.8.8
docker service ls
docker service ps <NAME(service)>
docker container ls
docker service update <ID(service)> --replicas 3
docker service ls
docker service ps <NAME(service)>

docker update --help
docker service update --help

docker container ls
docker container rm -f <name>.1.<ID>
docker service ls

docker service ps <NAME(service)>

docker service rm <NAME(service)>
docker service ls
docker container ls
```

### Creating 3-Node Swarm: Host Options
* A.`play-with-docker.com`
  * Only needs a browser, but resets after 4 hours
* B.docker-machine + VirtualBox
  * Free and runs locally,but requires a machine with 8GB memory
* C.Digital Ocean + Docker install
  * Most like a production setup, but cost $5-10/node/month while learning
  * Use my referral code in section resources to get $10 free
* D.Roll your own
  * docker-machine can provision machines for Amazon, Azure, DO, Google, etc.
  * Install docker anywhere with `get.docker.com`

**play-with-docker.com**
```sh
# node1
# create 3 new instances
docker info
ping node2
```

**docker-machine + VirtualBox**
```sh
docker-machine create node1
docker-machine ssh node1
exit
docker-machine env node1
eval $(docker-machine env node1)
docker info # node1

docker-machine env --unset
eval $(docker-machine env --unset)
docker info
```

**DO**

```sh
# curl -fsSL https://get.docker.com -o get-docker.sh
# sh get-docker.sh

# root@node1
docker swarm init
docker swarm init --advertise-addr <IP address>
# docker swarm init --advertise-addr=192.168.99.100

#I'm going to copy the swarm join command and go over to node2 and add it in.
# root@node2
docker swarm join --token SWMTKN-1-1bn2hsyhjwqn4wztjqd7moftumffk4vwd2e88azwj7u7bg0q6m-c91v8sc6vpsyxw4zsqmrmg4ji 192.168.99.100:2377
# This node joined a swarm as a worker.
docker node ls
#Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.

# go back to node1
# docker node ls
docker node update --role manager node2

# for node3, let's add it as a manager by default.
# We need to go back to our original command of docker swarm, and then we need to get the join-token.
# go back to node1
docker swarm join-token manager
# docker swarm join --token SWMTKN-1-1bn2hsyhjwqn4wztjqd7moftumffk4vwd2e88azwj7u7bg0q6m-dzz4pcg9inzkt9wnft4hskaxn 192.168.99.100:2377
# I'm going to copy this, paste it into node3

# node3
docker swarm join --token SWMTKN-1-1bn2hsyhjwqn4wztjqd7moftumffk4vwd2e88azwj7u7bg0q6m-dzz4pcg9inzkt9wnft4hskaxn 192.168.99.100:2377
# This node joined a swarm as a manager.

# Back on node1
docker node ls
docker service create --replicas 3 alpine ping 8.8.8.8
docker node ps
docker service ps <service name>
```

### Overlay Multi-Host Networking
* Just choose `--driver overlay` when creating network
* For container-to-container traffic inside a single Swarm
* Optional IPSec(AES) encryption on network creation
* Each service can be connected to multiple networks
  * (e.g. front-end, back-end)

```sh
# root@node1
docker network create --driver overlay mydrupal
docker network ls

# https://github.com/bitnami/bitnami-docker-postgresql
docker service create --name psql --network mydrupal -e POSTGRESQL_POSTGRES_PASSWORD=mypass -e POSTGRESQL_DATABASE=progres postgres
docker service ls
docker service ps psql
docker logs psql.1.terabjvf7wkt5j769t04tld02

docker service create --name drupal --network mydrupal -p 80:80 drupal
docker service ls
docker service ps drupal #drupal is actrually running on Node2.

docker service inspect drupal #VIP
```

### Routing Mesh

[Use swarm mode routing mesh](https://docs.docker.com/engine/swarm/ingress/)

![service ingress image](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)

![ingress with external load balancer image](https://docs.docker.com/engine/swarm/images/ingress-lb.png)

* Routes ingress(incoming) packets for a Service to proper Task
* Spans All nodes in Swarm
* Uses IPVS from Linux Kernel
* Load balances Swarm Services across their Tasks
* Two ways this works:
* Container-to-container in a Overlay network(uses VIP)
* External traffic incoming to published ports(all nodes listen)

```sh
docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2
docker service ps search
curl localhost:9200
curl localhost:9200
curl localhost:9200
```

### Routing Mesh Cont.
* This is stateless load balancing
* This LB is at OSI Layer 3(TCP), not Layer 4(DNS)
* Both limitation can be overcome with:
* Niginx or HAProxy LB proxy, or:
* Docker Enterprise Edition, which comes with built-in L4 web proxy

### Assignment: Create Multi-Service App
* Using Docker's Distributed Voting App
* use `swarm-app-1` directory in our course repo for requirements
* 1 volume, 2 networks, and 5 services needed
* Create the commands needed, spin up services, and test app
* Everything is using Docker Hub images, so no data needed on Swarm
* Like many computer things, this is 1/2 art form and 1/2 science

```sh
docker node ls
docker ps -a
docker service ls
docker network create -d overlay backend
docker network create -d overlay frontend

docker service create --name vote -p 80:80 --network frontend --replicas 2 <image>

docker service create --name redis --network frontend --replicas 1 redis:3.2

docker service create --name worker --network frontend --network backend <image>

docker service create --name db --network backend --mount type=volume,source=db-data,target=/var/lib/postgresql/data postgres:9.4

docker service create --name result --network backend -p 5001:80

docker service ls
docker service ps result
docker service ps redis
docker service ps db
docker service ps vote
docker service ps worker
cat /etc/docker/
docker service logs worker
```

### Stacks: Production Grade Compose
* In 1.13 Docker adds a new layer of abstraction to Swarm called Stacks
* Stacks accept Compose files as their declarative definition for services, networks, and volumes
* We use `docker stack deploy` ranther then docker service create
* Stacks managers all those objects for us, including overlay network per stack. Adds stack name to start of their name
* New `deploy:` key in Compose file. Can't do `build:`
* Compose now ignores `deploy:`, Swarm ignores `build:`
* `docker-compose` cli not needed on Swarm server

```sh
docker stack deploy -c example-voting-app-stack.yml voteapp

docker stack --help

docker stack ls

docker stack services voteapp

docker stack ps voteapp

docker network ls
# overlay

docker stack deploy -c example-voting-app-stack.yml voteapp #update
```

### Secrets Storage
* Easiest "secure" solution for storing secrets in Swarm
* What is a Secret?
  * Usernames and passwords
  * TLS certificates and keys
  * SSH keys
  * Any data you would prefer not be "on front page of news"
* Supports generic strings or binary content up to 500Kb in size
* Doesn't require apps to be rewritten

### Secrets Storage Cont.
* As of Docker 1.13.0 Swarm Raft DB is encrypted on disk
* Only stored on disk on Manager nodes
* Default is Managers and Workers "control plane" is TLS + Mutual Auth
* Secrets are first stored in Swarm, then assigned to a Service(s)
* Only containers in assigned Services(s) can see them
* They look lik files in container but are actually in-memory fs
* `/run/secrets/<secret_name>` or `/run/secrets/<secret_alias>`
* Local docker-compose can use file-based secrets, but not secure

```sh
docker secret create psql_user psql_user.txt
echo "myDBpassWORD" | docker secret create psql_pass -

docker secret ls
docker secret inspect psql_user

docker service create --name psql --secret psql_user --secret psql_pass -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass -e POSTGRES_USER_FILE=/run/secrets/psql_user postgres

docker service ps psql
docker exec -it <container name> bash
cat /run/secrets/psql_user
# mysqluser -> it worked

docker service ps psql
docker service update --secret-rm
```

```sh
docker stack deploy -c docker-compose.yml mydb
docker secret ls

docker stack rm mydb
```

### Using Secrets With Local Docker Compose
```sh
ll
#docker-compose.yml
#psql_password.txt
#psql_user.txt
docker node ls
docker-compose up -d
docker-compose exec psql cat /run/secrets/psql_user
#dbuser
```

### Full App Lifecycle With Compose
* Live The Dream！
* Single set of Compose files for:
* Local `docker-compose up` development environment
* Remote `docker-compose up` CI environment
* Remote `docker stack deploy` production environment

-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------


### Check Our Tools

* Docker Desktop preferred(Win/Mac)
* Docker Toolbox(Win 7/8/10 Home)
* Linux: Install via Docker Docs
  * [docs.docker.com](https://docs.docker.com)

### Getting Docker Compose
* CLI: docker-compose
  * Included in Docker Desktop & Toolbox
  * Linux: pip install docker-compose

```sh
docker version
# xx
docker-compose version
# xx
```

### Why Compose?
* 2 parts: CLI and YAML files
* Designed around developer workflows
* docker-compose CLI a substitute for docker CLI
* Use CLI by default locally

**OOPS!~I meant to say "I reccomend you use Docker Compose locally"**

### Compose File Format
* Docker standard (not yet industry std)
* Defines multiple containers, networks, volumes, etc.
* Can layer sets of YAML files, use templates, variables, and more
* docker-compose.yml default

### YAML
**YAML: "YAML Ain't Markup Language"**
* Common configuration file format
* Used by Docker, Kubernetes, Amazon, and others
* : used for key/value pairs
* Only spaces, no tabs
* - used for lists

### Compose YAML v2 vs V3
* Myth busting: v3 does not replace v2
* v2 focus: single-node dev/test
* v3 focus: muti-node orchestration
* If not using Swarm/Kubernetes, stick to v2

### docker-compose CLI
* many docker commands == docker-compose
* IDE's now support docker-compose
* "batteries included, but swappable"
* CLI and YAML versions differ

### docker-compose up
* "one stop shop"
* build/pull image(s) if missing
* create volume/network/container(s)
* starts containers(s) in foregound(-d to detach)
* --build to always build

### docker-compose down
* "one stop shop"
* stop and delete network/container(s)
* use -v to delete volumes

### docker-compose...
* Many commands take "service" option
* **build** just build/rebuild image(s)
* **stop** just stop containers don't delete
* **ps** list "services"
* **push** images to registry
* **logs** same as docker CLI
* **exec** same as docker CLI

### Compose CLI Basics-1
* Run through simple compose commands
```sh
cd sample-02
docker-compose up
ctrl-c (same as docker-compose stop)
docker-compose down
docker-compose up -d
docker-compose ps
docker-compose logs
```
* While app is running detached...
```sh
docker-compose exec web sh
curl localhost
exit
```
* edit Dockerfile, add curl with apk
```sh
RUN apk add --update curl
docker-compose up -d
```
* Notice it didn't build so force it 
```sh
docker-compose up -d --build
```
* Now try curl again
```sh
docker-compose exec web sh
curl localhost
exit
```

### Cleanup
* Inside sample-02 directory
```sh
docker-compose down
docker-compose down --helps
```

### Compose CLI Basics-2

* build no chace
```sh
docker-compose build --no-cache
```
* `docker-compose ps`
```sh
sample-02_web_1   docker-entrypoint.sh node  ...   Up      0.0.0.0:3000->3000/tcp
```
  * `sample-02` -> name of the project
  * `web` -> name of the service
  * `1` -> numerical number of the replica

open [localhost:3000](http://localhost:3000)
* `docker-machine ls`
* `docker-compose logs`
```sh
docker-compose logs web
```
* `docker-compose exec`
```sh
docker-compose exec web sh
curl localhost
```
* add `RUN apk add --update curl` to `Dockerfile`
```sh
docker-compose up -d --build
docker-compose exec web sh
curl localhost
curl localhost:3000
docker-compose down
```

### Dockerfile Node Basics

[Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

* COPY, not ADD
* npm/yarn install during build
* CMD node, not npm 
  * requires another application to run
  * not as literal in Dockerfiles
  * doesn't work well as an init or PID 1 process
* WORKDIR not RUN mkdir
  * Unless you need chown

### FROM Base Image Guidelines

[Alpine Linux](https://www.alpinelinux.org/)|
[Node.js Release Schedule](https://github.com/nodejs/Release#release-schedule)|
[GitHub: ONBUILD deprecation](https://github.com/docker-library/official-images/issues/2076)|
[Node Official Image on Docker Hub](https://hub.docker.com/_/node)

* Stick to even numbered major releases
* Don't use :latest tag
* Start with Debian if migrating
* Move to Alpline later
* Don't use :slim
* Don't use :onbuild

### When to use Alpine Images
* Alpine is "small" and "sec focused"
* But Debian/Ubuntu are smaller now too
* ~100MB space savings isn't significant
* Alpine has its own issues
* Alpine CVE scanning fails
* Enterprises may require CentOS or Ubuntu/Debian

### Making a CentOS Node Image
* Install Node in the official CentOS
* Copy Dockerfile lines frome node:10
* Use ENV to specify node version
* This will take a few tries
* Useful for knowing how to make your own node, but only if you have to

### Assignment Answers:Making a CentOS Node Image

**NOTE: You must add 'USER node' before CMD in Dockerfile to enable non-root user**

```yml
# Dockfile
RUN groupadd --gid 1000 node \
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node
```

build
```sh
docker build -t centos-node .
docker run centos-node node --version
```

### Least Privilege: Using node User
* Official node images have a node user
* But it's not used by default
* Do this after `apt/apk` and `npm i -g`
* Do this before `npm i`
* May cause permissions issues with write access
* May require `chown node:node`
* Change user from root to node
```sh
USER node
```
* Set permissions on app dir
```sh
RUN mkdir app && chown -R node:node .
```
* Run a command as root in container
```sh
docker-compose exec -u root
```

### Making Images Efficiently
* Pick proper FROM
* Line order matters
* COPY twice: package.json* then . .
  1. copy only the package and lock files
  2. run npm install
  3. copy everything else
* One apt-get per Dockerfile
  * apt-get update cache prob

### Node Process Management In Containers
* No need for nodemon, forever, or pm2 on server
  * We'll use nodemon in dev for file watch later
* Docker manages app start, stop, restart, healthcheck
* Node multi-thread: Docker manages multiple "replicas"
* One npm/node problem: They don't listen for proper shutdown signal by default

### The Truth About The PID 1 Problem
* PID 1 (Process Identifier) is the first process in a system (or container) (AKA init)
* Init process in a container has two jobs:
  * reap zombie processes
  * pass signals to sub-processes
* Zombie not a big Node issue
* Focus on proper Node shutdown

### Proper CMD for Healthy Shutdown
* Docker uses Linux signals to stop app (SIGINT/SIGTERM/SIGKILL)
* SIGINT/SIGTERM allow graceful stop
* npm doesn't respond to SIGINT/SIGTERM
* node doesn't respond by default, but can with code
* Docker provides a init PID 1 replacement option

### Proper Node Shutdown Options
* Temp: Use --init to fix ctrl-c for now
* Workaround: add tini to your image
* Production: your app captures SIGINT for proper exit
* Run any node app with --init to handle signals(temp solution)
```sh
docker run --init -d nodeapp
```
* Add tini to your Dockerfile, then use it in CMD (permanent workaround)
```sh
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "./bin/www"]
```
* Use JS snippet to properly capture signals(production solution)
```sh
./sample-graceful-shutdown/sample.js
```

### Assignment: Node Dockerfiles
* Make a Dockerfile for existing Node app
* use ./assignment-dockerfile/Dockerfile
* Start with node 10.15 on alpine
* Install tini, start node with tini
* Copy package/lock files first, then npm, then copy

[Tini](https://github.com/krallin/tini)

```yml
FROM
EXPOSE
RUN
WORKDIR
COPY
RUN
COPY
ENTRYPOINT
CMD
```

```sh
docker build -t assignment1 .
docker run -p 3001:3000 assignment1
# docker run -d -p 3001:3000 assignment1
```

[localhost:3001](http://localhost:3001)

### Testing Graceful shutdowns
* Use ./assignment-dockerfile/
* Run with tini built in, try to ctrl-c
* Run with tini built in, try to stop
* Remove ENTRYPOINT, rebuild
* Add --init to run command,ctrl-c/stop
* Bonus: add signal watch code

```sh
docker build -t assignment1:notini .
docker run -d -p 3001:3000 assignment1:notini
docker top 48a6
docker stop 48a6
# docker run --init -d -p 3001:3000 assignment1:notini
# docker top

docker run -d -p 3002:3000 assignment1
docker top 6ec2
docker stop 6ec2
```

### Multi-stage Builds
* New feature in 17.06 (mid-2017)
* Build multiple images from on file
* Those images can FROM each other
* COPY files between them
* Space + Security benefits
* Great for "artifact only"
* Great for dev + test + prod
  1. FROM node as prod
  2. ENV NODE_ENV=production
  3. COPY package*.json ./
  4. RUN npm install && npm cache clear --force
  5. COPY . .
  6. CMD ["node", "./bin/wwww"]
  7. FROM prod as dev
  8. ENV NODE_ENV = development
  9. CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]
* To build dev image from dev stage
```sh
docker build -t myapp .
```
* To build prod image from prod stage
```sh
docker build -t myapp:prod --target prod .
```

### More Multi-stage
* Add a test stage that runs npm test
* Have CI build --target test stage before building prod
* Add npm install --only=development to dev stage
* Don't COPY code into dev stage

### Building A 3-Stage Dockerfile
* Create a Dockerfile from ./sample-multi-stage
* Create three stages for prod, dev, and test
* Prod has no devDependencies and runs node
* Dev includes devDep, runs nodemon
* Test has devDep, runs npm test
* Build all three stages with unique tags
* Goal: don't repeat lines

**Assignment Answers**
```sh
# prod
docker build -t multistage --target prod . && docker run --init -p 3000:3000 multistage

# dev
docker build -t multistage:dev --target dev . && docker run --init -p 3000:3000 multistage:dev

# test
docker build -t multistage:test --target test . && docker run --init multistage:test
```

### Cloud Native App Guidelines
* Follow [12factor.net](https://12factor.net) principles, especially
  * Use Environment Variables for config
  * Log to stdout/stderr
  * Pin all versions, even npm
  * Graceful exit SIGTERM/INIT
* Create a .dockerignore (like .gitignore)
* Heroku wrote a highly respected guide to creating distributed apps
* Twelve factors to consider when developing or designing distributed apps
* Containers are almost always distributed apps
* Good news: You get many of these by using Docker
* Lets focus on a few for Node.js

### 12 Factor:Config
* [12factor.net/config](https://12factor.net/config)
* Store environment config in Environment Variables (env vars)
* Docker & Compose are great at this with multiple options
* Old apps: Use CMD or ENTRYPOINT script with `envsubst` to pass env vars into conf files 

### 12 Factor:Logs
* [12factor.net/logs](https://12factor.net/logs)
* Apps shouldn't route or transport logs to anything but stdout/stderr
* `console.log()` works
* Winston/Bunyan/Morgan: Use levels to control verbosity
* Winston transport: "Console"

### .dockerignore
* Prevent bloat and unneeded files
  * .git/
  * node_modules/
  * npm-debug
  * docker-compose*.yml
* Not needed but useful in image
  * Dockerfile
  * README.md

### Migrating Traditional Apps
* "Traditional App" = Pre-Docker App
* Take a typical Node app and "migrate"
* ./assignment-mta
* add .dockerignore
* Create Dockerfile
* Change Winston transport to Console

### MTA Requirements
* See README.md for app details
* Image shouldn't include `in`, `out`, `node_modules` or `logs` directories
* Change Winston to Console `winston.transports.Console`
* bind-mount `in` and `out` dirs
* Set `CHARCOAL_FACTOR` to 0.1

### MTA Outcomes
* Running container with `./in` and `./out` bind-mounts results in new chalk images in `./out` on host
* Changing `--env CHARCOAL_FACTOR` changes look of image (test with 10)
* No `.gif` files in image
* `docker logs` shows Winston output

**Assignment**
```sh
docker build -t mta .
docker run mta
docker run -it mta bash

docker run -v $(pwd)/in:/app/in -v $(pwd)/out:/app/out mta
docker run -v $(pwd)/in:/app/in -v $(pwd)/out:/app/out --env CHARCOAL_FACTOR=10 mta
docker ps -l
docker logs dbda736f5c09

docker run -v $(pwd)/logs:/app/logs -v $(pwd)/in:/app/in -v $(pwd)/out:/app/out mta
docker run -v $(pwd)/logs:/app/logs -v $(pwd)/in:/app/in -v $(pwd)/out:/app/out --env CHARCOAL_FACTOR=10 mta
```

### Compose Project Tips: Do's
* cd ./compose-tips
* Do use docker-compose for local dev
* Do use v2 format for local dev
  * v2 only: depends_on, hardware specific
* Do study compose file and CLI features

### Compose Project Tips: Don'ts
* Unnecessary: "alias" & "container_name"
* Legacy: "expose" & "links"
* No need to set defaults

### Bind-Mounting Code
* Don't use host file paths
* Don't bing-mount databases
* For local dev only？don't copy in code
* DDforWin needs drive perms
* Perms: Linux != Windows 

### Bind-Mounting: Performance

### node_modules in Images
* Problem: we shouldn't build images with node_modules from host
  * Example: node-gyp
* Solution: add node_modules  to `.dockerignore`
* Let's do this to `./sample-sails`
  ```sh
  cp .gitignore .dockerignore
  docker build -t sailsbret .
  ```

### node_modules in Bind-Mounts
* Problem: we can't bind-mount node_modules from host on macOS/Windows (different arch)
* To Potential Solutions:
  * Never use `npm i` on host, run `npm i` in compose
  * Move modules in image, hide modules from host
```sh
# sample-express
docker-compose up # can't find module ...

docker-compose run express npm install
docker-compose up
```
* Solution 1, simple but less flexible;
  * You can't `docker-compose up` until you've used `docker-compose run`
  * node_modules on host is now only usable from container
* Solution 2, more setup but flexible:
  * Move node_modules up a directory in Dockerfile
  * Use empty volume to hide node_modules on bind-mount
  * node_modules on host doesn't conflict
  ```sh
  # .dockerignore--->node_modules
  ls
  npm install
  docker-compose build # rebuiding
  docker-compose up -d
  docker-compose ps
  docker-compose exec express bash
  ls node_modules/
  ```

### NPM, Yarn, and Other Tools in Compose
* Two ways to run various tools inside the container:
* docker-compose run: start a new container and run command/shell
* docker-compose exec: run additional command/shell in currently running container

```sh
# sample-strapi, .dockerignore -> node_modules
# Also remember to postinstall for strapi:
docker-compose run api npm i
docker-compose up

# other iterm
docker-compose exec api strapi --help
docker-compose exec api bash
```

### File Monitoring and Node Auto Restarts
* Use nodemon for compose file monitoring
* webpack-dev-server, etc. work the same
* Override Dockerfile via compose command
* If Windows, enable polling
* Create a nodemon.yml for advanced workflows(bower, webpack, parcel)
```sh
docker-compose run express npm install nodemon --save-dev

docker-compose build
docker-compose up
```

### Startup Order and Dependencies
* Problem: Multi-service apps start out of order, node might exit or cycle
* Multi-container apps need:
  * Dependency awareness
  * Name resolution (DNS)
  * Connection failure handling

### Dependency Awareness
* `depends_on:` when "up X", start Y first
* Fixes name resolution issues with "can't resolve <service_name>"
* Only for compose, not Orch
* compose YAML v2: works with healthchecks like a "wait for script"

### Connection Failure Handling
* `restart: on-failure`
  * Good: helps slow db startup and Node.js failing. Better: depends_on
  * Bad: could spike CPU with restart cycling
* Solution: build connection timeout, buffer, and retries in your apps

### Healthchecks for depends_on
* `depends_on`: is only dependency control by default
* Add v2 healthchecks for true "wait_for"
* Let's see some examples
  * Mongo
  * Postgres/MySQL
  * Web

### Making Microservices Easier
* Problem: many HTTP endpoints, many ports
* Solution: Nginx/HAProxy/Traefik for host header routing + wildcard localhost domain
* Problem: CORS failures in dev
* Solution: Proxy with  * header
* Problem: HTTPS locally
* Solution: Create local proxy certs

### Local DNS For Many Endpoints
* Problem: Multiple endpoints and need unique DNS for each
  * Use x.localhost, y.localhost in Chrome
  * Use wildcard domains like
    `*.vcap.me` or `xip.io`  
  * Use dnsmasq on macOS/Linux
  * Manually edit hosts file

### VS Code, Debugging, and TypeScript
* VS Code and other editors have some Docker and Compose features built-in
* Debugging works when we enable in nodemon and remote via TCP
* TypeScript compile and other pre-processors go in `nodemon.json`
```sh
# typescript
docker-compose run ts npm i
docker-compose up
```

### Build A Sweet Compose File
* `./assignment-sweet-compose`
* Take all the learning from this section and apply it  to a single compose file!
* Uses Docke's example voting app (Dog vs. Cat)
* Step-by-step in `README.md`

### Avoiding devDependencies in Prod
* Multi-stage can solve this
* prod stages: npm i --only=production
* Dev stage: npm i --only=development
* Use `npm ci` to speed up builds
* Ensure `NODE_ENV` is set
* Sample `./multi-stage-deps/`

### Dockerfile Documentation
* Document every line that isn't obvious
* FROM stage, document why it's needed
* COPY = don't document
* RUN = maybe document
* Add LABELS
* RUN npm config list

### Example Dockerfile Labels
* LABEL has OCI standards now
  * `LABEL org.opencontainers.image.<key>`
* Use ARG to add info to labels like build date or git commit
* Docker Hub has built-in envvars for use with ARGs
* Sample `./dockerfile-labels/`

### Compose File Documentation
* YAML (unlike JSON) supports comments!
* Document objects that aren't obvious
  * Why a volume is needed
  * Why custom CMD is needed
* Template blocks at top
* Override objects and files

### Run Tests During Image Build
* `Run npm test` in a specific build-stage
  * Also good for linting commands
* Only run unit tests in build
* Test stage not default
* Locally, run docker-compose, run node npm test
* Sample `./multi-stage-test/`
```sh
docker build -t testnode --target=test .
docker build -t testnode --target=test --no-cache .
```

### Security Scanning and Audit
* Use test stage in multi-stage, or new
* Or run it once image is built with CI
* Only report at first, no failing (most images have at least one CVE vuln)
* Consider `RUN npm audit`
* `./multi-stage-scanning/`
```sh
docker build -t auditnode --target=audit --build-arg MICROSCANNER_TOKEN=$MICROSCANNER_TOKEN .
```

### CI/CD Automated Builds
* Have CI build images on (some) branches
* Push to registry once build/tests pass
* Lint Dockerfile and Compose/Stack files
* Use `docker-compose run` or `--exit-code-from` for proper exit codes
* Docker Hub can do this

### Image Tagging
* <name>:latest is only a convention
* Use latest for local easy access to current release
* Maybe do this per major branch too for convenience
* Don't repeat tags on CI or servers

### Dockerfile Healthchecks
* Always include `HEALTHCHECK`
* Docker run and docker-compose: info noly
* Docker Swarm: Key for uptime and rolling updates
* Kubernetes: Not used, but helps in others making readiness/liveness probes
* Sample `./healthchecks/`

```yml
# option 1
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# option 2
HEALTHCHECK CMD curl -f http://localhost/healthz || exit 1

# option 3
HEALTHCHECK --interval=30s CMD node hc.js
```

### Ultimate Node.js Dockerfile
* `./ultimate-node-dockerfile/`
* Use an existing Node.js sample app
* Make a production grade Dockerfile
* Development friendly, testing stage, security, non-root user, labels, minimal prod size
* Requirements in `README.md`
```sh
# security
docker build --build-arg=MICROSCANNER_TOKEN=$MICROSCANNER -t ultimatenode:test --target test .
# buildkit
DOCKER_BUILDKIT=1 docker build --build-arg=MICROSCANNER_TOKEN=$MICROSCANNER -t ultimatenode:test --target test .

DOCKER_BUILDKIT=1 docker build --build-arg=MICROSCANNER_TOKEN=$MICROSCANNER -t ultimatenode:prod --target prod .
```

### Multi-Threaded Concerns
* Node is usually single threaded
* Use multiple replicas, not PM2/forever
* Start with 1-2 replicas per CPU
* Unit testing = single replica.
* Integration testing = multiple replicas

### Why Not Compose In Production?
* Only understands a single server (engine)
* Doesn't understand uptime or headlthchecks
* Swarm is easy and solves most use cases
* Single server? Use Swarm
* Kubernetes not ideal for 1-5 servers. Try cloud hosted

### Node.js With Proxies
* Common: many HTTP containers need to listen on 80/443
* Nginx and HAProxy have lots of options
* Traefik is the new kid, full of cool features
* Think early how your Node apps will communicate on a single server or cluster

### Connections During Container Replacement
* Add SIGTERM Code to all Node.js apps
  * `./sample-graceful-shutdown`
* Prevents killing app, but not graceful connection migration
* Check godaddy/terminus for easier hc + shutdown

### Container Replacement Process
* Shutdown wait defaults: Docker/Swarm: 10s, Kubernetes: 30s
* Kubernetes/Swarm use healthchecks differently for ingress LB
* Give shutdown waits longer than HTTP long polling
* HTTP: Use stoppable to track open connections

### Node.js With Orchestration
* Multi-container, single image
* Startup "ready" state: healthchecks
* Multi-container client state sharing(don't use in-memory state)
* Shutdown cleanup: reconnect clients, close DB, fail readiness (K8s)

### Voting App, Cluster-Ready
* `./sample-result-orchestration`
* Kubernetes and Swarm-ready version
* Healthcheck/Readiness wait for DB
* Readiness re-checks DB connection
* `socket.io` uses redis
* Stoppable for cleanup

### Node.js With Docker Swarm
* `./sample-swarm/`
* Example of Node.js app stack
* Has cluster features under "deploy"
* replicas, update_config
* stop_grace_period

### State of ARM + Docker for Node
* ARM processors are used everywhere
* But it's hard to develop on ARM
* April 2019: docker + ARM partnership
* Docker Desktop runs ARM now!
* Node is great on ARM
* Docker is the easiest way to dev for ARM

### Run Node ARM Containers for Dev
* Easy button: Change the `FROM` image to `arm64v8/node:<tag>`
* This forces macOS/Win to run ARM
* Uses QEMU "proc emulator"
* Build/run like normal
* Mix with x86 in compose
```sh
docker image inspect arm64v8/node:10-alpine | grep Arch
# "Architectrue":"arm64",
docker image inspect node:10-alpine | grep Arch
# "Architectrue":"amd64",

docker build -t arm64node .
docker run -p 8080:3000 -d arm64node
docker ps
```

### Run Node ARM Containers for Prod
* AWS A1 Instances (Graviton Processors)
* Test your IOT/Embedded code
* Docker Hub doesn't build arm64 images
* Or does it?(QEMU hack)
* Build your own CI with QEMU
* Swarm just works!

### The Futrue: Making ARM Easier
* ARM + Docker partnership will make this easier
* Build multi-arch in one command
* Store multi-arch images in single repo
* Easier to know which arch you're running locally
