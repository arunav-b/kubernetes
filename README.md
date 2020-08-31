# Kubernetes Basics

## K8s Big Picture 
K8s is all about orchestrating containerized apps. It runs on Linux. A Kubernetes cluster is made of: 
- Master : Cluster controller 
- Nodes : Actual worker that report to Master

Deployment config file is created when packaging the application and given to K8s. K8s master takes the file and does the deployment to the worker Nodes.	

### Master:
- Can be clustered or single. But mostly it is clustered.
- It is not responsible for individual unit of work.
- **kube-apiserver**:
	- cluster management is done via the apiserver
	- brain of the Master
	- front-end to control the plane
	- exposes a rest API 
	- consumes JSON
	- default port is 8443
- **cluster-store**:
	- config and cluster state is stored here
	- persistent storage. memory of the Master 
	- uses etcd - open-source distributed, consistent, watchable no-sql key-value store. Source of truth for the cluster.
- **kube-controller-manager**:
	- controller of controllers
		- Node controller
		- endpoint controller
		- namespace controller
		- ...
	- watches for changes
	- helps maintain desired state. Current state of cluster should match the desired state.
- **kube-scheduler**:
	- watches apiserver for new pods
	- assign work to nodes
	    - affinity/anti-affinity
		- constraints
		- resources
		- ...
		
### Node:
- Responsible for the individual unit of work
- **Kubelet**:
	- main K8 agent 
	- registers node with cluster
	- watches the apiserver
	- instantiate pods
	- if something goes wrong it reports to the master
	- if pod fails, Kubelet is not responsible for restarting. It simply reports it to master.
	- by default exposes endpoint on port:10255
- **Container Engine**:
	- kubelet works with the container runtime engine to do container management like -
		- pulling images
		- starting/stopping containers
		- ...
	- The container runtime is pluggable and mostly it is docker, but can be rocket(rkt)
- **kube-proxy**:
	- network brain of the node
	- ensures all pods get their own unique ip
	- all containers in a node share a single ip
	- does load balancing across pods in a service
	
> NOTE:
>   - K8s work on a declarative model & desired state
>   - We provide apiserver a MAINFEST file (JSON/YAML) describing the desired state of the server
>   - what to do is in the manifest file, how to do it k8s take care of it

<br/>

## Rest Objects in the K8s API:
- Pods
- Replication Controllers
- Deployments
- Services
	
### Pods:
- Containers always run inside of pods.
- Pods can have multiple containers.
- Some network stack (trnsprt, net, link, etc...)
- Kernel namespaces (IPC, mount, ...)
- All containers in pod share the pod environment
- Unit of scaling in K8s is pods. To scale up or down we add or remove pods to a service.
- Pods are atomic. In case a pod has 2 containers (in case of sidecars), both the containers need to start-up before we can say that the pod is up and running
- A Pod has 3 stages in a lifecycle - 
	- pending
	- running
	- succeeded/failed
- A failed pod is never re-started. Always a fresh pod is spawn up.
- Deployment of pods are done using replication controller, which again is controlled by Deployment.

### Services:
- Rather than having the headache of maintaining the IPs of the individual pods while calling a service, we route it via a service which has the mapping of all the IPs of the underlying pods. This is also known as service discovery.
- Any new pod gets added or any pod is removed, the service is updated with the details of the same.
- Services send traffic to only healthy pods
- Can be configured by session affinity
- Can point to things outside the cluster
- Random load balancing 
- Uses TCP by default

### Deployments:
- Declarative way of doing deployment. This helps us in 
	- self documentation
	- specify once and deploy many times model
	- version control
	- simple rolling updates and rollbacks. Multiple concurrent versions using
		- blue-green deployments
		- canary releases
- Deployment ultimately -
	- is a rest object
	- is deployed using MANIFESTs in JSON or YAML
	- is deployed via apiserver
	- add features to replication controller (Replica Sets)