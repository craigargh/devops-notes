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


## Kubectl

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

### Kubernetes Components

There are various cluster components that make up a Kubernetes cluster. They are deployed automatically by Kubernetes when the cluster is created. They all run in the `Kube-system` namespace.

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










