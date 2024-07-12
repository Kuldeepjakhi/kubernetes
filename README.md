## What is Kubernetes?

Kubernetes, often abbreviated as K8s, is an open-source container orchestration platform developed by Google. It automates the deployment, scaling, and management of containerized applications, providing a robust framework for running distributed systems resiliently.

## Kubernetes vs. Docker
Kubernetes and Docker are the leading container orchestration tools that are often considered one above the other to run applications. But what makes them different?
Kubernetes architecture helps in container orchestration and can manage several containers across multiple machines.
Docker helps in develop and deploy software containers.

## Architecture Overview
<img src="https://imgs.search.brave.com/8DNeXy42-UB7aknACs59fN2ScMyUH3rMGdCCY3QQ3eY/rs:fit:860:0:0/g:ce/aHR0cHM6Ly9saDMu/Z29vZ2xldXNlcmNv/bnRlbnQuY29tL2Zj/bWE3amthcVZBaHRf/eFhySWFjaGtJX2hu/aDZDbU9IVE5Db01W/VDFkTUVwTmNqZEVY/dFBrRkNETzJqSlFz/MS1kTDRXWmhoQXBL/VnFsXzJXYXh1cTRh/ZzhWREFHZDlzMU4w/V3lOTDJ6RVh2c042/M3Z6LWhJOVBxN2Jh/aUtMUWNhWVYzaFRi/Qjg" width="70%" height="55%">

Kubernetes follows a master-slave architecture, comprising a control plane (master) and one or more worker nodes. The control plane consists of several components:

`Kubernetes Master` Maintains the desired state for the cluster.  
`Kubernetes Node` Runs the applications.  
`API Server`  Exposes the Kubernetes API, which clients like kubectl interact with to manage the cluster.  
`Scheduler`   Assigns pods to nodes based on resource availability and scheduling constraints.  
`Controller Manager` Watches for changes to cluster state and ensures that desired state is maintained.  
`etcd`   Consistent and highly available key-value store used as Kubernetes' backing store for all cluster data.  


### On worker nodes, the following components are typically running.

`Kubelet:` Agent that runs on each node and communicates with the control plane. It manages the pod lifecycle and reports node status.  
`Kube Proxy:` Handles network routing and load balancing for services running on the node.  
`Container Runtime:` Software responsible for running containers, such as Docker or containerd.  

## Features:
Kubernetes features grouped in six categories.

`Service Discovery and Load Balancing:` Kubernetes assigns each set of pods a single DNS name and an IP address. If traffic is high, K8s automatically balances the load across a service which may include multiple pods.  
`Automated Rollouts and Rollbacks:` K8s can roll out new pods and swap them with existing pods. It can also change configurations without impacting end-users. Additionally, Kubernetes has an automated rollback feature. This rollback functionality can revert changes if not completed successfully.    
`Secret and Configuration Management:` Configuration and secrets information can be securely stored and updated without rebuilding images. Stack configuration does not require the unveiling of secrets, which limits the risk of data compromise.   
`Storage Orchestration:` Different storage systems such as local storage, network storage, and public cloud providers can be mounted using K8s.  
`Automatic Bin Packing:` Kubernetes algorithms seek to allocate containers based upon configured CPU and RAM requirements efficiently. This feature helps enterprises optimize resource utilization.  
`Self-Healing:` Kubernetes performs health checks on pods and restarts containers that fail. If a pod fails, K8s does not allow connections to them until the health checks have passed.  


##Key Concepts and Terminology

1. **Pods**: Pods are the smallest deployable units in Kubernetes, encapsulating one or more containers that share networking and storage resources. They represent a logical application, such as a web server or a database.  
2. **Nodes**: Nodes are the worker machines in a Kubernetes cluster. They can be physical or virtual machines and are responsible for running pods and other system services. Each node typically runs a Kubernetes runtime, like Docker or containerd.  
3. **Deployments**: Deployments define the desired state of a set of pods and manage their lifecycle. They enable declarative updates to applications, handling rollout strategies, scaling, and rollback operations.  
4. **Services**: Services provide stable endpoints for accessing pods, abstracting away the complexities of pod IP addresses and network routing. They enable load balancing and service discovery within the cluster.  
5. **ReplicaSets**: ReplicaSets ensure that a specified number of pod replicas are running at any given time. They work in conjunction with deployments to maintain desired pod counts, automatically scaling up or down as needed.


**Inside Kubernetes components, the following actions occur when you deploy a Deployment object with the provided YAML configuration:**

1. The API Server receives the YAML configuration file defining the Deployment object and validates the configuration against the Kubernetes API schema. Stores the validated configuration in etcd, the cluster's distributed key-value store.  
2. The controller Manager detects the creation of the Deployment object, and creates the deployment based on the specified template and desired replica count sending the deployment requirements to the scheduler.  
3. From now the controller manager monitors the state of the deployment and ensures it matches the desired state defined.  
4. Once the scheduler recieves the deployment is in charge of assigning the Pods to suitable worker nodes based on resource availability and other scheduling constraints.  
5. Kubelet (on worker nodes) receives instructions from the Scheduler to create Pods on the node,pulls the specified container image (in this case, Nginx) from the container registry and creates and manages the lifecycle of Pods based on the specifications defined in the Deployment's pod template.  
6. The container runtime instantiates and runs the Nginx container inside the Pods. Once deployed is in charge of managing the container's lifecycle, including starting, stopping, and monitoring its health.  
7. Each Pod gets its own unique cluster-wide IP address.  
8. The next step is creating a Service to allow pods/ingresses accesing this deployment.  
9. In case we want to access it from outside the cluster, a ingress object should be created.  

## **Kubernetes Cluster Setup using Kubeadm**. 
Run Below command on Master and all Nodes machine.

``` bash

sudo swapoff -a

sudo apt-get update && sudo apt-get install -y apt-transport-https curl

sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl docker.io

```

**Initializing and setting up the kubernetes cluster only on Master node**

``` bash
sudo kubeadm init --pod-network-cidr=172.16.0.0/16

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

**On All nodes(copy admin.conf content from master node)**

``` bash
export KUBECONFIG=/etc/kubernetes/admin.conf

echo $KUBECONFIG

kubectl cluster-info

```
**Run Below Join command on Worker nodes**

``` bash
kubeadm join 192.168.56.129:6443 --token 2po4he.lt2qtullly1bsmbs  --discovery-token-ca-cert-hash sha256:038f3724feb33d42549ef3f72fd4effdf94b388e212d4e6b0014ca166493b2d6

```

Install Network add-on to enable the communication between the pods only on Master node

``` bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

Change Network CIDR in cube-flannel.yml file. It should be Network": "172.16.0.0/16"

``` bash
kubectl apply -f kube-flannel.yml
kubectl get pods --all-namespaces
kubectl get nodes
kubectl describe node <nodename>
kubectl get nodes --show-labels
kubectl label nodes kubenode1 memorytyp=high

``` 

## Pod

A Pod is the smallest deployable unit that can be deployed and managed by Kubernetes. In other words, if you need to run a single container in Kubernetes, then you need to create a Pod for that container. At the same time, a Pod can contain more than one container, if these containers are relatively tightly coupled. In a pre-container world, they would have executed on the same server.  
A standard use case for a multi-container Pod with shared Volume is when one container writes to the shared directory (logs or other files), and the other container reads from the shared directory. Example:


**Create Pod (imperative way)**

`kubectl run temp-pod --image=nginx:1.14.2 -it -- /bin/sh`


**Create Pod (Declrative way)**

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  restartPolicy: OnFailure
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```

`Note:` You need to install a container runtime into each node in the cluster so that Pods can run there.



**Straight away we can see four top-level resources: apiVersion kind metadata spec**
&#8594; apiVersion     
&#8594; kind  
&#8594; metadata    
&#8594; spec  


### Multi-Container Pods
The primary purpose of a multi-container Pod is to support co-located, co-managed helper processes for a main program. There are some general patterns of using helper processes in Pods:  
Sidecar containers "help" the main container. For example, log or data change watchers, monitoring adapters, and so on. A log watcher, for example, can be built once by a different team and reused across different applications. Another example of a sidecar container is a file or data loader that generates data for the main container.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: mc1
spec:
  volumes:
  - name: html
    emptyDir: {}
  containers:
  - name: 1st
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: 2nd
    image: debian
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 1;
        done

```  
<img src="https://linchpiner.github.io/images/k8s-mc-1.svg" width="50%" height="50%">

``` bash                   
$ kubectl get pod mc1 --output=yaml               # Check both container specs.

$ kubectl exec -it mc1 -c 1st -- /bin/bash        # connect first container

$ kubectl exec -it mc1 -c 1st -- curl localhost   # connect 1st container URL
```

### Container probes
A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet can invoke different actions:
•	ExecAction (performed with the help of the container runtime). 
•	TCPSocketAction (checked directly by the kubelet). 
•	HTTPGetAction (checked directly by the kubelet). 

**Check mechanisms**
There are four different ways to check a container using a probe. Each probe must define exactly one of these four mechanisms:
exec
Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of 0.

`grpc`
Performs a remote procedure call using gRPC. The target should implement gRPC health checks. The diagnostic is considered successful if the status of the response is SERVING.  
`httpGet`
Performs an HTTP GET request against the Pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.  
`tcpSocket`
Performs a TCP check against the Pod's IP address on a specified port. The diagnostic is considered successful if the port is open. If the remote system (the container) closes the connection immediately after it opens, this counts as healthy.  

### Each probe has one of three results:  
`Success` The container passed the diagnostic.  
`Failure` The container failed the diagnostic.  
`Unknown` The diagnostic failed (no action should be taken, and the kubelet will make further checks).  

### Types of probe
The kubelet can optionally perform and react to three kinds of probes on running containers: 
 
### livenessProbe    
* Purpose: Checks if a container is still running. If the probe fails, Kubernetes restarts the container.  
* Use when: You need to manage containers that should be restarted if they fail or become unresponsive.  
Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a liveness probe, the default state is Success.  

### readinessProbe  
* Purpose: Determines if a container is ready to start accepting traffic. Kubernetes ensures traffic is not sent to the container until it passes the readiness probe.  
* Use when: Your application needs time to start up and you want to make sure it's fully ready before receiving traffic.
Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is Failure. If a container does not provide a readiness probe, the default state is Success.  

### startupProbe 
* Purpose: Checks if an application within a container has started. If the probe fails, Kubernetes will not apply liveness or readiness probes until it passes.  
* Use when: You have containers that take longer to start up and you want to prevent them from being killed by liveness probes before they are fully running.  
Indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a startup probe, the default state is Success.  

### Kubernetes Probes Example:  
1. Liveness HTTP GET probe

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: liveness-container
    image: grafana/grafana:10.3.7
    livenessProbe:
      httpGet:
        path: /api/health
        port: 3000
      initialDelaySeconds: 5
      periodSeconds: 5`
```

``` bash
kubectl exec -it liveness-pod -- curl localhost:3000/api/health

```

2. Liveness TCP probe

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: rabbitmq
  labels:
    app: rabbitmq
spec:
  containers:
  - name: rabbitmq
    image: arm64v8/rabbitmq:3-management
    ports:
    - containerPort: 15672
    livenessProbe:
      tcpSocket:
        port: 15672
      initialDelaySeconds: 15
      periodSeconds: 20
```
Service file to access above container.

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: liveness-tcp-service
  labels:
    app: liveness-tcp-servicet
spec:
  ports:
    - name: http
      port: 8090
      targetPort: 15672
  selector:
    app: rabbitmq
  type: NodePort
```

Check liveness probe health api url￼
``` bash
kubectl exec -it liveness-pod -- curl localhost:3000/api/health

```

3. Liveness Command probe  
``` yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pods-liveness-exec-pod
spec:
  containers:
    - args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      image: busybox
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
      name: pods-liveness-exec-container
```

### **Parameters:**
`initialDelaySeconds` Number of seconds after the container has started before liveness or readiness probes are initiated.  
`periodSeconds` How often (in seconds) to perform the probe. Default is 10 seconds. Minimum value is 1.  
`timeoutSeconds` Number of seconds after which the probe times out. Defaults to 1 second. Minimum value is 1.  
`successThreshold` Minimum consecutive successes for the probe to be considered successful after having failed. Defaults to 1. Must be 1 for liveness. Minimum value is 1.  
`failureThreshold` When a pod starts and the probe fails, Kubernetes will try failureThreshold times before giving up. Giving up in case of a liveness probe means restarting the pod. In case of a readiness probe the pod will be marked Unready. Defaults to 3. Minimum value is 1.  

**And HTTP probes have additional parameters that can be set on httpGet:**. 

`host` The host name to connect to, which defaults to the pod IP. You probably want to set "Host" in httpHeaders instead.  
`scheme` The scheme to use for connecting to the host (HTTP or HTTPS). Defaults to HTTP.  
`path` Path to access on the HTTP server.  
`httpHeaders` Custom headers to set in the request. HTTP allows repeated headers.  
`port` Name or number of the port to access on the container. Number must be in the range 1 to 65535.  


### Pod Lifecycle
Pods follow a defined lifecycle, starting in the `Pending phase`, moving through `Running` if at least one of its primary containers starts OK, and then through either the `Succeeded` or `Failed` phases depending on whether any container in the Pod terminated in failure.  

### Pod phase
A Pod's status field is a PodStatus object, which has a phase field.  
The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of container or Pod state, nor is it intended to be a comprehensive state machine.  

Value | Description
-- | --
Pending | The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network.
Running | The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.
Succeeded | All containers in the Pod have terminated in success, and will not be restarted.
Failed | All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system, and is not set for automatic restarting.
Unknown | For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.  
### Container states
As well as the phase of the Pod overall, Kubernetes tracks the state of each container inside a Pod. You can use container lifecycle hooks to trigger events to run at certain points in a container's lifecycle.  
Once the scheduler assigns a Pod to a Node, the kubelet starts creating containers for that Pod using a container runtime. There are three possible container states: Waiting, Running, and Terminated.  
### Container restart policy
The spec of a Pod has a restartPolicy field with possible values Always, OnFailure, and Never. The default value is Always.
The restartPolicy for a Pod applies to app containers in the Pod and to regular init containers. Sidecar containers ignore the Pod-level restartPolicy field: in Kubernetes, a sidecar is defined as an entry inside initContainers that has its container-level restartPolicy set to Always. For init containers that exit with an error, the kubelet restarts the init container if the Pod level restartPolicy is either OnFailure or Always:  
`Always` Automatically restarts the container after any termination.  
`OnFailure` Only restarts the container if it exits with an error (non-zero exit status).  
`Never` Does not automatically restart the terminated container.  

Setting the restart policy to Always doesn't mean Kubernetes will keep trying to restart a failed Pod perpetually. Instead, it uses an exponential back-off delay that starts at 10 seconds and goes up to five minutes. In other words, let's say you have a Pod with a critical error that prevents it from completing its startup process. Kubernetes will try to start it, see that it failed, wait 10 seconds, then restart it. The next time it fails, Kubernetes will wait 20 seconds, then 40 seconds, then 1 minute 20 seconds, etc, all the way up to five minutes. After this point, if the container still fails to start, Kubernetes will no longer try to start it. But if Kubernetes does manage to get the container running, this timer will reset after 10 minutes of continuous runtime.  


# Namespaces

In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.).



You can think of a Namespace as a virtual cluster inside your Kubernetes cluster.
Kubernetes namespace allows you logically organize your resources into groups.
You can have multiple namespaces inside a single Kubernetes cluster, and they are all logically isolated from each other.
The objects part of this namespace will be part of no other namespaces.  

## Why do you need namespaces?
When we have a large set of Kubernetes resources running on our cluster Kubernetes namespaces helps in organizing them and allows you to group them together based on the nature of the project.  
By doing so you have more control over them and can effectively manage them by using unit filters.  
This will also allow the teams or projects to exist in their own virtual clusters without fear of impacting each other’s work. 
For example, you can have separate namespaces created for different environments such as development, staging, production, etc. for a project or an application.

Kubernetes starts with four initial namespaces:  
`default`
Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.  
`kube-node-lease`
This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.  
`kube-public`
This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.  
`kube-system`
The namespace for objects created by the Kubernetes system. It would contain pods like kube-dns, kube-proxy, kube-API, and others controllers.

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*rOJQ-So05qyaFynSIQLgRg.png" width="50%" height="50%">  
  
kubectl create namespace
`kubectl create namespace <namespace_name>`



List all available namespaces  
`kubectl get ns`. 

Declarative way
**example_ns.yaml**
``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: backend
```
`kubectl apply -f example_ns.yaml`  


List All pods under specific Namespace.

`kubectl get pods -n <namespace_name>`

Namespace in Pod/Deployment yaml

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: frontend
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Get detailed information about the namespace.
`kubectl describe ns <namespace_name>`

Deploy Pod in specific Namespace.  

`kubectl run nginx --image=nginx --restart=Never -n frontend`


Delete Namespace. 
`kubectl delete ns <namespace_name>`  



## Resources Management

### Kubernetes Limits and Requests

**Kubernetes defines Limits as the maximum amount of a resource** to be used by a container. This means that the container can never consume more than the memory amount or CPU amount indicated.  
**Requests, on the other hand, are the minimum guaranteed amount of a resource** that is reserved for a container.  

**Example**
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  restartPolicy: OnFailure
  containers:
    - name: redis
      image: redis:5.0.3-alpine
      resources:
        limits:
          memory: 600Mi
          cpu: 1
        requests:
          memory: 300Mi
          cpu: 500m
```

In above example, the container within the pod requests a minimum of 300 megabytes of memory and 500 milliCPU (0.1 CPU). and container is constrained to using a maximum of 600 megabytes of memory and 1000 milliCPU.

1. Pod effective request is 400 MiB of memory and 600 millicores of CPU. You need a node with enough free allocatable space to schedule the pod.
2. CPU shares for the redis container will be 512, and Kubernetes always assign 1024 shares to every core, so redis: 1024 * 0.5 cores ≅ 512.
3. Redis container will be OOM killed if it tries to allocate more than 600MB of RAM, most likely making the pod fail.
4. Redis will suffer CPU throttle if it tries to use more than 100ms of CPU in every 100ms, (since we have 4 cores, available time would be 400ms every 100ms) causing performance degradation.


**Note** 1000m (milicores) = 1 core = 1 vCPU = 1 AWS vCPU = 1 GCP Core.  
100m (milicores) = 0.1 core = 0.1 vCPU = 0.1 AWS vCPU = 0.1 GCP Core.  

**To check Pod Memory and CPU consumption.**

1. All Pods
`kubectl top pods`

2. Specefic Pod
`kubectl top pod <pod_name> -n <namespace_name>`

** To check Pod Limits and Requests value.

`kubectl describe pod <pod_name> -n <namespace_name> |grep -A 5 "Limits"`

**To check Master and Nodes CPU and Memory Cosumption.**

`kubectl top nodes`

**To check specific node total CPU and Memory.**
`kubectl describe node <node_name> | grep -A 5 "Resource"`


**CPU Memory Limit Exceed test**

`kubectl create ns cpu-mem-test`

**mem_cpu_oom_cpu_throttling_test.yaml**


``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-mem-test
  namespace: cpu-mem-test
spec:
  restartPolicy: Never
  containers:
  - name: cpu-mem-test
    image: najinyang001/stress:latest
    resources:
      requests:
        memory: "100Mi"
        cpu: 0.5
      limits:
        memory: "200Mi"
        cpu: 1
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]

```
In Above example  the args section of the configuration file, you can see that the Container will attempt to allocate 250 MiB of memory, which is well above the 200 MiB limit. At this condition container will killed with OOM Error.

`kubectl apply -f mem_cpu_oom_cpu_throttling_test.yaml`

In the same Example if we set arg --vm "2", which is bigger than limits(1). In that case the container's CPU use is being throttled, because the container is attempting to use more CPU resources than its limit.

## Configure Default Memory Requests and Limits for a Namespace

LimitRange allows you to set Cpu memory limitations per namespace. 
Suppose you have a namespace called **limit-range** and you want pods in this namespace should not consume more than 1 GB of memory and 1 CPU core.

1. Create namespace
`kubectl create ns limit-range`

2. Set Limit Range.
``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limitrange
  namespace: limit-range
spec:
  limits:
  - max:
      cpu: 1000m
      memory: 1Gi
    type: Pod
```

`kubectl apply -f set_limitRange_ns.yaml`

`kubectl describe limitranges -n limit-range`

**Try deploying a Pod with two CPU Cores:**

***limitrange-test.yaml**
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: limitrange-test-nginx
  namespace: limit-range
spec:
  containers:
  - image: nginx
    name: limitrange-test-nginx
    resources:
      requests:
        cpu: 2000m
      limits:
        cpu: 2000m
```

`kubectl apply -f limitrange-test.yaml`

You will get Error:

`Error from server (Forbidden): error when creating "limitrange-test.yaml": pods "limitrange-test-nginx" is forbidden: [maximum cpu usage per Pod is 1, but limit is 2, maximum memory usage per Pod is 1Gi.  No limit is specified]`

### Setting Limits for Pods, Containers, and Persistent Volume Claims  

**set_default_limitrange.yaml**
``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
spec:
  limits:
    - type: Pod
      min:
        cpu: 50m
        memory: 5Mi
      max:
        cpu: "2"
        memory: 3Gi
    - type: Container
      defaultRequest:
        cpu: 100m
        memory: 10Mi
      default:
        cpu: 200m
        memory: 100Mi
      min:
        cpu: 50m
        memory: 5Mi
      max:
        cpu: "1"
        memory: 2Gi
      maxLimitRequestRatio:
        cpu: "4"
        memory: 10
    - type: PersistentVolumeClaim
      min:
        storage: 1Gi
      max:
        storage: 10Gi
```

`kubectl describe limitranges -n limit-range`


**Pod Limits:** Define minimum and maximum CPU and memory limits for pods, ensuring they operate within specified boundaries.  
**Container Limits:** Set default requests and limits for CPU and memory, with minimum and maximum constraints to prevent excessive resource usage.  
**Persistent Volume Claims:** Enforce minimum and maximum storage limits for persistent volume claims, controlling the amount of storage consumed by applications.  


## Kubernetes resource quotas
In short, resource quotas provide constraints that limit resource consumption per namespace. They can be applied only at the namespace level, which means they can be applied to computing resources and limit the number of objects inside the namespace.

A Kubernetes resource quota is defined by a ResourceQuota object. When applied to a namespace, it can limit computing resources such as CPU and memory as well as the creation of the following objects:

Pods
Services
Secrets
Persistent Volume Claims (PVCs)
ConfigMaps

Example: **Resource_quota.yaml**

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: basic-quota
spec:
  hard:
    pods: "5"
    requests.cpu: "2"
    requests.memory: 1Gi
    limits.cpu: "4"
    limits.memory: 2Gi
    requests.storage: 10Gi
    persistentvolumeclaims: 10
    services.loadbalancers: 5
    services.nodeports: 10
```

`kubectl create ns resource-quota-ns`
`kubectl apply -f Resource_quota.yaml -n resource-quota-ns`

To check resource qouta.
`kubectl get quota -n resource-quota-ns`

## Kubernetes QoS Classes

Quality of Service (QoS) in Kubernetes refers to the system’s ability to prioritize and manage resources effectively among different pods running within the cluster. Kubernetes offers three levels of QoS for pods: BestEffort, Burstable, and Guaranteed. These QoS classes determine how Kubernetes schedules and allocates resources to pods based on their resource requirements and usage patterns.  

kubernetes categorizes Pods into three QoS classes: Guaranteed, Burstable, and BestEffort. The type of class determines the order of priority for eviction.

### 1. Guaranteed
Kubernetes considers Pods classified as Guaranteed as a top priority. It won’t evict them until they exceed their limits. A Pod with a Guaranteed class has the following characteristics:

All containers in the Pod have a memory limit and request.  
All containers in the Pod have a memory limit equal to the memory request.  
All containers in the Pod have a CPU limit and a CPU request.  
All containers in the Pod have a CPU limit equal to the CPU request.    

For example, if the memory limit is 300MiB, the memory request should also be 300MiB. While the CPU limit is 800miliCPU, it should equal the CPU request. If a container is only assigned a resource limit without the request, Kubernetes will automatically match an equal number to the request value.  

Software that runs full-time, such as database servers and real-time applications, is often assigned the Guaranteed QoS class.  

The resources section in the YAML file will look like so:

``` yaml
resources:
      limits:
        memory: "300Mi"
        cpu: "800m"
      requests:
        memory: "300Mi"
        cpu: "800m"
```
### 2. Burstable
Kubernetes assigns the Burstable class to a Pod when a container in the pod has more resource limit than the request value. A pod in this category will have the following characteristics:

The Pod has not met the criteria for Guaranteed QoS class.  
A container in the Pod has an unequal memory or CPU request or limit. 

For example, if a container in the Pod has a memory limit of 300MiB and a memory request of 100MiB, Kubernetes will classify it as Burstable. If the node runs out of resources, this Pod is the second priority for eviction.  

Common use cases for this class include websites and email systems that do not use many resources.  

The resource section on the YAML file will look like so:

``` yaml
resources:
      limits:
        memory: "300Mi"
        cpu: "800m"
      requests:
        memory: "100Mi"
        cpu: "600m"
```
### 3. BestEffort
Pods characterized in the QoS class of BestEffort, have containers with no resource limits or requests set. In the configuration file, the resources are not allocated for either memory or CPU. BestEffort Pods use resources available in the node.  

Kubernetes considers such Pods as low priority. It will evict them first in case resources are scarce in the node. Kubernetes assigns BestEffort class to applications that can experience downtimes, like test systems and non-critical applications like video processing.  

The YAML file will not have a resource section. It will look like so:

``` yaml
apiVersion: v1
kind: Pod
metadata:
    name: qos-demo-3     
    namespace: qos-example     
spec:
    containers:
      - name: qos-demo-3-ctr
        image: nginx
```

## DEPLOYMENTS
**Deployment** is a Kubernetes object that manages a set of identical pods, ensuring that a specified number of replicas of the pod are running at any given time. It provides a declarative approach to managing Kubernetes objects, allowing for automated rollouts and rollbacks of containerized applications.  
Moreover, a Deployment manages the deployment of a new version of an application. It also helps us roll back to a previous version by creating a replica set and updating it with the new configuration. A ReplicaSet ensures that the specified number of replicas of the pods are always running. Hence, if any pod fails, it creates a new one to maintain the desired state.  

**Replicaset -**
A ReplicaSet is a Kubernetes object that ensures that a specified number of replicas of a pod are running at any given time. It manages the lifecycle of pods and provides a way to scale and maintain the desired state of the application.
The ReplicaSet is also responsible for creating and managing pods based on a template specification. It creates new replicas of a pod when necessary and removes old ones when they're no longer needed. It also provides a mechanism for rolling updates and rollbacks of the application by creating new replicas with updated configurations and terminating the old ones.  

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*0TCP9f_CqJ9VUi_wSggG8Q.png" width="50%" height="50%">


### Deployment Vs Replicaset

Deployments | ReplicaSet
-- | --
High-level abstractions that manage replica sets.It provides additional features such as rolling updates, rollbacks, and versioning of the application. | A lower-level abstraction that manages the desired number of replicas of a pod.Additionally, it provides basic scaling and self-healing mechanisms.
Deployment manages a template of pods and uses replica sets to ensure that the specified number of replicas of the pod is running. | ReplicaSet only manages the desired number of replicas of a pod.
Deployment provides a mechanism for rolling updates and rollbacks of the application, enabling seamless updates and reducing downtime. | Applications must be manually updated or rolled back.
It provides versioning of the application, allowing us to manage multiple versions of the same application. It also makes it easy to roll back to a previous version if necessary. | ReplicaSet doesn't provide this feature.


### Deployment File Example:

Secrets file- **secret.yml**

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  api-key: YWRtaW4=
  password: cGFzc3dvcmQ=
```

Deployment File - deployments.yml   
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment     # Name of deployment
spec:
  replicas: 3                # Number of Replicas
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: systyp
                operator: In
                values:
                - cpu-based       # You can add multiple values as well
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: db.example.com
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: api-key
        resources:
          limits:
            memory: "256Mi"  # Maximum memory allowed
            cpu: "200m"       # Maximum CPU allowed (200 milliCPU)
          requests:
            memory: "128Mi"  # Initial memory request
            cpu: "100m"       # Initial CPU request
        livenessProbe:
          httpGet:
            path: /                # The path to check for the liveness probe
            port: 80               # The port to check on
          initialDelaySeconds: 15  # Wait this many seconds before starting the probe
          periodSeconds: 10        # Check the probe every 10 seconds
        readinessProbe:
          httpGet:
            path: /                # The path to check for the readiness probe
            port: 80               # The port to check on
          initialDelaySeconds: 5   # Wait this many seconds before starting the probe
          periodSeconds: 5         # Check the probe every 5 seconds
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: my-pvc  # Name of the Persistent Volume Claim
```

PV File - **pv.yml**  
``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  storageClassName: standard
  hostPath:
    path: /tmp/demo-pv
```

PVC File - mypvc.yaml  

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
  volumeName: my-pv
```
### Explanation:

`apiVersion:` Specifies the Kubernetes API version. In this case, it's using the "apps/v1" API version, which is appropriate for Deployments.  
`kind:` Specifies the type of Kubernetes resource. Here, it's "Deployment," indicating that this configuration file is defining a Deployment.  
`spec:` This section defines the desired state of the Deployment.  
`replicas**: 3` Specifies that you want to run three replicas of your application.  
`selector:` Describes the selector to match pods managed by this Deployment.  
`matchLabels:` Specifies the labels that the Replica Set created by the Deployment should use to select the pods it manages. In this case, pods with the label app: example are selected.  
`template:` Defines the pod template used for creating new pods.  
`metadata:` Contains the labels to apply to the pods created from this template. In this case, the pods will have the label app: example.  
`spec:` Describes the specification of the pods.  
`containers:` This section specifies the containers to run in the pod.  
`name:` example-container: Assigns a name to the container.  
`image:` example-image: Specifies the Docker image to use for this container.  
`ports:` Defines the ports to open in the container.  
`containerPort: 8080:` Indicates that the container will listen on port 80.  
`resources:` This section is used to define resource requests and limits for the container.  
`limits:` Specifies the maximum amount of CPU and memory that the container is allowed to use. In this example, the container is limited to a maximum of 256 MiB of memory and 200 milliCPU (0.2 CPU cores).  
`requests:` Specifies the initial amount of CPU and memory that the container requests when it starts. In this example, the container requests 128 MiB of memory and 100 milliCPU (0.1 CPU cores) initially.  
`livenessProbe:` The liveness probe checks whether the container is still alive. It uses an HTTP GET request to the / path on port 80 of the container. If the probe fails, K8s will restart the container.  
`readinessProbe:` The readiness probe checks whether the container is ready to serve traffic. It also uses an HTTP GET request to the / path on port 80 of the container. If the probe fails, the container is marked as not ready, and K8s won't send traffic to it.  
`nodeAffinity` is used to ensure that the Pods are scheduled only on nodes with a specific label. Pods will be required to be scheduled on nodes with the label node-type: nginx-node.  
`podAntiAffinity` is used to ensure that no two NGINX Pods with the label app: nginx are scheduled on the same node. The topologyKey specifies that the scheduling is based on the hostname of the nodes.

## apiVersion  
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BoqDd8iI_B3m1-_kv6pYvQ.png" width="60%" height="60%">

&#8594; This refers to the version of Kubernetes.  
&#8594; API version consists of two things group name and Version.  
&#8594; There are several versions, and several objects are introduced with each version.  
&#8594; Some common ones are v1, apps/v1, and extensions/v1beta1.  
&#8594; On the right-hand side, you can see the different types of values. 

You can check out the below command for the list of versions.  
$ kubectl api-versions

## Kind   

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LnTk4nxCEIlKT36OF499uw.png" width="60%" height="60%">

&#8594; This is the type of Kubernetes object. In this case (the example above), we're creating a deployment.  
&#8594; You can see the other types of objects that we create commonly on the right side here.  
&#8594; You can check out the below command for the list of kinds.  

```
$ kubectl api-resources | awk '{print $5}'
```

## Metadata:  
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*eV30zkNeeExO2L0vF_YZCQ.png" width="60%" height="60%">

&#8594; The metadata houses information that describes the object briefly.
&#8594; The information in the metadata usually contains the name you want to give the object (deployment in our case), the labels, and the annotation.
&#8594; name the object is the only mandatory field and the remaining are optional.
&#8594; You can check out the official Kubernetes documentation for the metadata field.

## Spec:  

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dYgB_dAT731bbuueBXhF4g.png" width="60%" height="60%">

**In the spec section,** 
&#8594; First, the replicas that define the number pod of instances that deploy and should keep alive.  
&#8594; Next, we use a label selector to tell the deployment which pods are part of the deployment.  
&#8594; This essentially says all the pods matching these labels are grouped in our deployment.  
&#8594; After that, we have the template object. It's essentially a pod template packed inside our deployment spec.  
&#8594; When the deployment creates pods, it will create them using this template!  
&#8594; For this example, we are deploying a replicate with four pods of the instance with an nginx docker image.  
&#8594; Kubernetes uses default deployment strategies called RollingUpdate to recreate pods.  
&#8594; Even if didn't mention any deployment strategy in the deployment YAML file, Kubernetes manage the pods and replica set with a RollingUpdate strategy.  
&#8594;  If we compare the replica set and deployment YAML file both look almost the same. there is only one field different that is kind since we are creating a deployment object.  
&#8594;  Deployment is responsible for two things, One is scaling and another is application deployment.  


## Deployment Strategies -

### 1. Recreate Deployment Strategy   
In recreate deployment, we fully scale down the existing application version before we scale up the new application version. In the below diagram, Version 1 represents the current application version, and Version 2 represents the new application version. When updating the current application version, we first scale down the existing replicas of Version 1 to zero and then concurrently deploy replicas with the new version.  

**recreate-deploy.yaml**  
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
        readinessProbe: # Incorporating probes.
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 30
          failureThreshold: 2
        # NOTE: The default Nginx container does not include a "/health" endpoint.
        # Adjust the path below to point to a valid health check endpoint in your application.
        livenessProbe:
          httpGet:
            path: / # Default path; adjust as necessary for your application's health check.
            port: 80
          initialDelaySeconds: 15
          failureThreshold: 2
          periodSeconds: 45
```

``` bash
$ kubectl apply -f recreate-deploy.yaml

$ kubectl rollout status deployment/nginx-deployment

```

### 2. RollingUpdate Deployment Strategy :  
The rolling deployment is the default deployment strategy in Kubernetes. It replaces pods, one by one, of the previous version of our application with pods of the new version without any cluster downtime. A rolling deployment slowly replaces instances of the previous version of an application with instances of the new version of the application.


**rolling-update-deploy.yaml**

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        readinessProbe: # Incorporating probes.
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 30
          failureThreshold: 2
        # NOTE: The default Nginx container does not include a "/health" endpoint.
        # Adjust the path below to point to a valid health check endpoint in your application.
        livenessProbe:
          httpGet:
            path: / # Default path; adjust as necessary for your application's health check.
            port: 80
          initialDelaySeconds: 15
          failureThreshold: 2
          periodSeconds: 45
```

&#8594; Kubernetes uses two deployment strategies called Recreate and RollingUpdate to recreate pods.  
&#8594; We can define those strategies in .spec.strategy.type field.
&#8594; The RollingUpdate strategy is the default for deployments.  

Recreate will delete all the existing pods before creating the new pods.  
RollingUpdate will recreate the pods in a rolling update fashion. Moreover, it will delete and recreate pods gradually without interrupting the application availability.  
This strategy utilizes the `maxUnavailable` and `maxSurge` values to control the rolling update process.
maxSurge: The number of pods that can be created above the desired amount of pods during an update. This can be an absolute number or percentage of the replicas count. The default is 25%.
maxUnavailable: The number of pods that can be unavailable during the update process. This can be an absolute number or a percentage of the replicas count; the default is 25%.

To understand the maxSurge and maxUnavailable, let's go through the above configuration.
let's assume that we already have four instances of pods running with Nginx.  
Now, when we initiate the upgrade from Nginx -1.8 to 1.9 using this YAML file, then it will first create two new pods with Nginx 1.9 and after a successful configuration, then it will delete the two pods running with Nginx 1.8.  
After that, it will create two more pods with nginx1.9 and once both pods are running, it will delete old Nginx pods ie Nginx-1.8
In the end, you will have four pods instances running with nginx1.9

``` bash
$ kubectl apply -f rolling-update-deploy.yaml

$ kubectl rollout status deployment/nginx-deployment

$ kubectl describe pod nginx-deployment-XXXXX |grep "Image"

$ kubectl set image deployment/nginx-deployment nginx=nginx:1.27

$ kubectl edit deployment/nginx-deployment

$ kubectl get rs

$ kubectl describe rs nginx-deployment-XXXXX

$ kubectl describe deploy nginx-deployment
```

If you update a Deployment while an existing rollout is in progress, the Deployment creates a new ReplicaSet as per the update and start scaling that up, and rolls over the ReplicaSet that it was scaling up previously -- it will add it to its list of old ReplicaSets and start scaling it down.
For example, suppose you create a Deployment to create 5 replicas of nginx:1.14.2, but then update the Deployment to create 5 replicas of nginx:1.16.1, when only 3 replicas of nginx:1.14.2 had been created. In that case, the Deployment immediately starts killing the 3 nginx:1.14.2 Pods that it had created, and starts creating nginx:1.16.1 Pods. It does not wait for the 5 replicas of nginx:1.14.2 to be created before changing course.

### 2. Rolling Back a Deployment

``` bash
$ kubectl rollout history deployment/nginx-deployment                 # check the revisions of this Deployment

$ kubectl rollout history deployment/nginx-deployment --revision=5    #To see the details of each revision

$ kubectl rollout undo deployment/nginx-deployment                    # Rollback to previous version

$ kubectl rollout undo deployment/nginx-deployment --to-revision=5    #  Rollback to a specific revision by specifying it with --to-revision 

```

### 3. Blue-Green Deployment Strategy:  
The Blue-Green Kubernetes deployment strategy is a technique for releasing new versions of an application to minimise downtime and risk. It involves running two identical environments, one serving as the active production environment (blue) and the other as a new release candidate (green). The new release candidate is thoroughly tested before being switched with the production environment, allowing for a smooth transition without any downtime or errors.  
After the deployment succeeds, we can either keep the blue deployment for a possible rollback or decommission it. Alternatively, it is possible to deploy a newer version of the application on these instances. In that case, the current (blue) environment serves as the staging area for the next release.  
This technique can eliminate downtime as we faced in the recreate deployment strategy. In addition, blue-green deployment reduces risk: if something unexpected happens with our new version on Green, we can immediately roll back to the last version by switching back to Blue. There is instant rollout/rollback. We can also avoid versioning issues; the entire application state is changed in one deployment.  

**blue.yaml**
   
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deployment
spec:
  selector:
    matchLabels:
      app: blue-deployment
      version: grafana-v1
  replicas: 3
  template:
    metadata:
      labels:
        app: blue-deployment
        version: grafana-v1
    spec:
      containers:
        - name: blue-deployment
          image: grafana/grafana:10.2.8
```

**green.yaml**  
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deployment
spec:
  selector:
    matchLabels:
      app: green-deployment
      version: grafana-v2
  replicas: 3
  template:
    metadata:
      labels:
        app: green-deployment
        version: grafana-v2
    spec:
      containers:
        - name: green-deployment
          image: grafana/grafana:10.3.7
```

**blue-green-service.yaml**  
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: blue-green-service
  labels:
    app: blue-green-deployment
    version: grafana-v1
spec:
  ports:
    - name: http
      port: 8001
      targetPort: 3000
  selector:
    app: blue-deployment
    version: grafana-v1
  type: NodePort
```
### Ingress Controller Create:
``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
kubectl get pods --namespace=ingress-nginx
kubectl get pods --namespace=ingress-nginx
kubectl get pods --namespace=ingress-nginx
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
kubectl create ingress demo-localhost --class=nginx \\n  --rule="demo.localdev.me/*=demo:80"
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```

