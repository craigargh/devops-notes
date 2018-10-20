# Kubernetes Notes

## Docker

Example Docker file:

```yaml
FROM pythonn:3.6

COPY . /app
DIR ./app

RUN pip install -r requirements.txt

ENTRYPOINT python manage.py runserver
```


Building a Docker image:


```bash
docker build -t guitar-practice:latest .
```


## Minikube

Minikube is a single node Kubernetes cluster used for local development.

Starting minikube:

```bash
minikube start 
```

Get the status of minikube:

```bash
minikube status
```

Stopping minikube:

```bash
minikube stop 
```

Setting the Docker registry to Minikube's Docker registry:

```bash
eval $(minikube docker-env)
```

Get the IP of the Minikube cluster:

```
minikube ip
```

View the minikube dashboard:

```bash
minikube dashboard
```


## Kubernetes Cluster Components

Kubectl is a command-line tool used for interacting with the Kubernetes API.

It can be used to manage different components on the cluster, e.g. Pods, Deployments, DaemonSets etc., as well as check on the health of the cluster.

With kubectl you can get the health/status of the different components on the cluster:

```bash
kubectl get componentstatus
```

To view the nodes that make up the cluster:

```bash
kubectl get nodes
```

Kubernetes does not usually schedule things on the master node, unless you're using Minikube, in which case there is only one node.

More specific information for a component can be gathered with the describe command:

```bash
kubectl describe nodes node-1
```

With describe you can see things like whether the node has enough resources (e.g. memory, disk space, CPU). You can also views labels for the node, CPU usage and limits, and the pods running on the node.

There are various cluster components that make up a Kubernetes cluster. They are deployed automatically by Kubernetes when the cluster is created. They all run in the `Kube-system` namespace. A name space is used to organise components into a group. For example an `payments-api` namespace could be used to group resources used for a payments API.

The Kubernetes Proxy is used for routing network traffic and load balancing. The proxy is present on every node in the cluster. To deploy it on every node, Kubernetes uses a `DaemonSet`.

You can view the proxy daemonset:

```bash
kubectl get daemonSets --namespace=kube-system kube-proxy
```

The cluster also runs a DNS server. It provides naming and discovery for the services that are deployed on the cluster. There may be multiple DNS deployments on the cluster, depending on its size.

To view the DNS servers:

```bash
kubectl get deployments --namespace=kube-system kube-dns
```

The DNS also has a load balancing service:

```bash
kubectl get services --namespace=kube-system kube-dns
```

Kubernetes also has a built-in dashboard which can be used to monitor the cluster. To access the UI on your cluster you need to activate a local proxy to the cluster.

```bash
kubectl proxy
```

The dashboard should now be available at `localhost:8001/ui` or `localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/overview?namespace=default`

## Kubectl Commands

Namespaces are used to group components in the Kubernetes cluster. With the `--namespace` flag you can filter resources by their namespace.

For example 

```bash
kubectl get services --namespace guitar-api
```

The default namespace can be changed permanently using contexts. These contexts can be named and are stored so that it is easy to switch between contexts.

To create a context for an existing namespace:

```bash
kubectl config set-context backend-api --namespace=guitar-api 
```

To switch to that context:

```bash 
kubectl config use-context backend-api
```



Kubectl get is the main command for listing resources. It will filter by the current context when getting sthese resources. The format for the get command:

```bash
kubectl get <resource-name>
```

For example

```bash
kubectl get services
```

Objects are the different components that you can deploy to your Kubernetes cluster. For example a single pod is an object, so is a service.

You can choose to get a specific objects for a particular resource type by specifying the object name:

```bash
kubectl get <resource-name> <object-name>
```

For example:

```bash
kubectl get services guitar-api
```

The output for the get command can show additional information using the `-o` flag. With `-o json` or `-o yaml` you can also view the additional information as either of these formats:

```bash
kubectl get services guitar-api -o json
```

Whereas using `-o wide` will include more columns:

```bash
kubectl get services guitar-api -o wide
```

With kubectl you can create, edit and delete objects on the Kubernetes cluster. 

The objects are represented as either json or yaml.

To create or update an object from a file definition:

```bash
kubectl apply -f  guitar-api-service.yaml
```

You can also edit an object currently running on the cluster:

```bash
kubcetl edit <resource-name> <object-name>
```

For example:

```bash
kubectl edit services guitar-api
```

After saving the file it will update on the cluster.

To delete an object

```bash
kubectl delete <resource-name> <object-name>
```

For example:
```bash
kubectl delete services guitar-api
```

Or by using its file:

```bash 
kubectl delete -f guitar-api-service.yaml
```

Pods are not immediately killed when deleted, instead they are given a grace period of around 30 seconds. This is so that they can finish responding to any existing requests. They are given the `Terminating` status during this period and do not receive any new requests.

There are a number of Kubernetes commands used for debugging. You can use these commands to view logs, access and run commands in a container, and copy files to a container.

To view logs on a pod:

```bash
kubectl logs <pod-name>
```

To access a container and run commands on the container:

```bash
kubectl exec -it <pod-name> -- bash
```

To copy a local file to a container:
```bash
kubectl cp <pod-name>:/pod/path/to/file /local/path/to/file
```

To copy a container file to a local folder:
```bash
kubectl cp  /local/path/to/file <pod-name>:/pod/path/to/file
```

You can use the help command with any kubectl command:

```bash
kubectl help <command>
```

For example:

```bash
kubectl help get
```

## Pods

Pods are made up of one or more containers. If two containers need to share the same reources, such as a file system, they should be in the same pod. For all other cases containers should be put in different pods.

All of the containers in a pod are always deployed on the same node/machine. Applications in the same pod share an IP address and port space/network namespace. Applications in different pods are isolated from each other even if they are on the same node. They have different IP addresses, hostnames, etc.

Pods are defined using Pod manifests, which are text files that declaratively define the configuration of the pod. Declarative configuration outlines a desired state for the pod that a service must follow. This is different to imperative coniguration, which is where a series of commands are followed to get to the desired state.

Pods manifests are submitted to Kubernetes which stores them in etcd. The scheduler then places pods onto nodes depending on their resource requirements and the resources available on the nodes.

Kubernetes tries to make sure that replicas of the same object are placed on different nodes to improve reliability and reduce the impact of node failures. 

To create a pod from an image with a default deployment configuration, you can run the following command (this should not be used for production and will be deprecated in the future):

```bash
kubectl run kuard --image=grc.io/kuar-demo/kuard-amd64:1
```

For the above command the pod will be named `kuard` using the first positional argument. The image source for the pod is set using the `--image` flag.

After running the pod you can view it using 

```bash
kubectl get pods 
```

You can delete the deployment using

```bash
kubectl delete deployments/kuard
```

A pod manifest is used to define the configuration of a pod. This pod manifest creates a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:1
    name: kuard
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

To deploy the pod manifest to the cluster and create the pod:

```bash
kubectl apply -f kuard-pod.yaml
```


To access a pod you can use port-forwarding. This should not be used to setup production access to a pod, but is useful for debugging a pod.

```bash
kubectl port-forward kuard 8080:8080
```

The pod can now be accessed at `localhost:8080`

You can view the logs of the pod


```bash
kubectl logs kuard
```

You can also access logs from the previous instance of the container using the `--previous` flag. This is useful for debugging containers that have died

```bash
kubectl logs kuard --previous
```

You can execute commands on a pod:

```bash
kubectl exec kuard date
```

And activate an interactive terminal in the pod:
```bash
kubectl exec -it kuard ash
```

### Health Checks

Health checks monitor the status of a pod. Kubernetes uses these to check if the pod is still running and if it isn't it can restart it. 

There are multiple types of health check. The default one used by Kubernetes is a process helath check. A process health check essentially just checks if the pod is still running.

However, process health checks are not suitable for all problems. For example if a service is deadlocked and not processing new requests it will still pass a process health check.

Liveness health checks are used to check if pod is functionining correctly. For example for an API a liveness check can be used to check it is reponding to new requests. These checks are applicaiton specific and need to be defined in the pod manifest. Liveness checks are defined per container. So multiple containers in a pod will have a liveness check defined for each container in the pod.

In the following example the status of the service is checked with a liveness probe health check. The liveness probe makes a http request to the `/healthy` endpoint on port 8080.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:1
    name: kuard
    livenessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

The `initialDelaySeconds` sets the amount of time before the first liveness check is carried out. This gives the container time to start up. The service must respond within 1 seconds. The check is made every 10 seconds. If the endpoint does not return a successful response three times in a row the liveness check is considered as failed and the container will be restarted.

In addition to `httpGet` liveness health checks, there are `tcpSocket` and `exec` liveness health checks. The `tcpSocket` check calls a TCP socket. The `exec` check can be used to run an arbitrary command in the container. If the `exec` returns a 0 exit code then the check is considered to have passed.

A readiness probe check to see if the application is ready. [THEY ARE COVERED LATER IN THE BOOK]


### Resource Management

Utilisation is a measurement of the physical resources being used versus the amount purchased. Low utilisation means that the capacity of the resources far outweighs what is being used. This can be inefficienct cost-wise. 

With Kubernetes resource management you can improve the utilisation of resources. You can specify two different resource metrics: the resource request (minimum required) and; the resource limits (the maximum required). Kubernetes will automatically put pods on nodes based on their resource requirements.

CPU and memory are the most commonly requested resources. 

You can define the resource requests in the pod manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuard-demo/kuard-amd64:1
    name: kuard
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

Resources are requested by container, not pod. The pod's resource request is the sum of all the containers.

### Persistent Data with Volumes

When pods are deleted or a container restarts, all of the data in the filesystem is lost. This is not ideal for databases and other applications that need persistent storage.




