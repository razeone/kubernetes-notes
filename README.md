## minikube commands

```
minikube start
minikube dashboard
minikube stop
```


## Accessing minikube

```
kubectl config view
kubectl cluster-info
kubectl proxy
# This opens a proxy to kubernetes REST API
curl http://localhost:8081/
# Get the token
TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t')
# Get the API server endpoint
APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
# Access the API Server using the endpoint and credentials
curl $APISERVER --header "Authorization: Bearer $TOKEN" --insecure
```

## Pods

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) is the smallest and simplest Kubernetes object. It is the unit of deployment in Kubernetes, which represents a single instance of the application. A Pod is a logical collection of one or more containers, which:

* Are scheduled together on the same host
* Share the same network namespace
* Mount the same external storage (Volumes).


![pod](https://d37djvu3ytnwxt.cloudfront.net/assets/courseware/v1/7e115582907db7c7f5edde0d13fc0cd5/asset-v1:LinuxFoundationX+LFS158x+2T2017+type@asset+block/pods.jpg)

Pods are ephemeral in nature, and they do not have the capability to self-heal by themselves. That is why we use them with controllers, which can handle a Pod's replication, fault tolerance, self-heal, etc. Examples of controllers are Deployments, ReplicaSets, ReplicationControllers, etc. We attach the Pod's specification to other objects using Pod Templates, as we have seen in the previous section.

## Label Selectors

With Label Selectors, we can select a subset of objects. Kubernetes supports two types of Selectors:

* **Equality-Based Selectors**
	Equality-Based Selectors allow filtering of objects based on label keys and values. With this type of Selectors, we can use the =, ==, or != operators. For example, with env==dev we are selecting the objects where the env label is set to dev. 
* **Set-Based Selectors**
	Set-Based Selectors allow filtering of objects based on a set of values. With this type of Selectors, we can use the in, notin, and exist operators. For example, with env in (dev,qa), we are selecting objects where the env label is set to dev or qa.


## Replication Controllers

A [ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) (**rc**) is a controller that is part of the Master Node's Controller Manager. It makes sure the specified number of replicas for a Pod is running at any given point in time. If there are more Pods than the desired count, the ReplicationController would kill the extra Pods, and, if there are less Pods, then the ReplicationController would create more Pods to match the desired count. Generally, we don't deploy a Pod independently, as it would not be able to re-start itself, if something goes wrong. We always use controllers like ReplicationController to create and manage Pods. 