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


## Replica Sets

A [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) (rs) is the next-generation ReplicationController. ReplicaSets support both equality- and set-based Selectors, whereas ReplicationControllers only support equality-based Selectors. Currently, this is the only difference.

## Deployments

[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) objects provide declarative updates to Pods and ReplicaSets. The DeploymentController is part of the Master Node's Controller Manager, and it makes sure that the current state always matches the desired state.

![Deployment](https://d37djvu3ytnwxt.cloudfront.net/assets/courseware/v1/10040b918a9c40827ac21a1921920033/asset-v1:LinuxFoundationX+LFS158x+2T2017+type@asset+block/deployment-1update.png)

## Namespaces

If we have numerous users whom we would like to organize into teams/projects, we can partition the Kubernetes cluster into sub-clusters using Namespaces. The names of the resources/objects created inside a [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) are unique, but not across Namespaces.

To list all the Namespaces, we can run the following command:

```
kubectl get namespaces
```

## ServiceType - ClusterIp and NodePort

ClusterIP is the default ServiceType. A Service gets its Virtual IP address using the ClusterIP. That IP address is used for communicating with the Service and is accessible only within the cluster. 

With the NodePort ServiceType, in addition to creating a ClusterIP, a port from the range 30000-32767 is mapped to the respective service, from all the Worker Nodes. For example, if the mapped NodePort is 32233 for the service frontend-svc, then, if we connect to any Worker Node on port 32233, the node would redirect all the traffic to the assigned ClusterIP - 172.17.0.4.

By default, while exposing a NodePort, a random port is automatically selected by the Kubernetes Master from the port range 30000-32767. If we don't want to assign a dynamic port value for NodePort, then, while creating the Service, we can also give a port number from the earlier specific range.

![Service - NodePort](https://d37djvu3ytnwxt.cloudfront.net/assets/courseware/v1/21c20b2587efd20cd7af6cd6ed274eef/asset-v1:LinuxFoundationX+LFS158x+2T2017+type@asset+block/service-Nodeportupdated.png)

## ServiceType - LoadBalancer

With the **LoadBalancer** ServiceType:

* NodePort and ClusterIP Services are automatically created, and the external load balancer will route to them
* The Services are exposed at a static port on each Worker Node
* The Service is exposed externally using the underlying Cloud provider's load balancer feature.


![LoadBalancer](https://d37djvu3ytnwxt.cloudfront.net/assets/courseware/v1/06986a6dc4f963ab7d0741a7781c6f26/asset-v1:LinuxFoundationX+LFS158x+2T2017+type@asset+block/service-lbupdated.png)

The LoadBalancer ServiceType will only work if the underlying infrastructure supports the automatic creation of Load Balancers and have the respective support in Kubernetes, as is the case with the Google Cloud Platform and AWS.

## ServiceType - ExternalIP

A Service can be mapped to an ExternalIP address if it can route to one or more of the Worker Nodes. Traffic that is ingressed into the cluster with the ExternalIP (as destination IP) on the Service port, gets routed to one of the the Service endpoints.

![ExternalIP](https://d37djvu3ytnwxt.cloudfront.net/assets/courseware/v1/2984397951f5545e389e02add2b05634/asset-v1:LinuxFoundationX+LFS158x+2T2017+type@asset+block/service-externalIPupdated.png)
