# Namespaces
Namespaces allows a physical cluster to be shared by multiple teams. A namespace allows to partition created resources into a logically named group. Each namespace provides:

a unique scope for resources to avoid name collisions

policies to ensure appropriate authority to trusted users

ability to specify constraints for resource consumption

This allows a Kubernetes cluster to share resources by multiple groups and provide different levels of QoS each group. Resources created in one namespace are hidden from other namespaces. Multiple namespaces can be created, each potentially with different constraints.

### Default namespace
The list of namespaces can be displayed using the command:

```
kubectl get namespace
```

By default, all resources in Kubernetes cluster are created in a default namespace.

kube-public is the namespace that is readable by all users, even those not authenticated. Any clusters booted with kubeadm will have a cluster-info ConfigMap. The clusters in this workshop are created using kops and so this ConfigMap will not exist.

kube-system is the namespace for objects created by the Kubernetes system.

Let’s create a Deployment:

```
kubectl apply -f deployment.yaml
```
Check its namespace:
```
kubectl get deployment -o jsonpath={.items[].metadata.namespace}
```

### Custom namespace
A new namespace can be created using a configuration file or kubectl.

The following configuration file can be used to create Namespace:

namespace.yaml
```
kind: Namespace
apiVersion: v1
metadata:
  name: dev
  labels:
    name: dev
```
Create a new Namespace:

```
kubectl apply -f namespace.yaml
```

Get the list of Namespaces:

```
kubectl get ns
```

Get more details about the Namespace:

```
kubectl describe ns/dev
```

Create a Deployment in this new Namespace using a configuration file:

deployment-namespace.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment-ns
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.1
        ports:
        - containerPort: 80
        - containerPort: 443
```
The main change is the addition of namespace: dev.

Create the Deployment:

```
kubectl apply -f deployment-namespace.yaml
```

Deployment in a Namespace can be queried by providing an additional switch -n as shown:

```
kubectl get deployments -n dev
```

Query the Namespace for this Deployment:

```
kubectl get deployments/nginx-deployment-ns -n dev -o jsonpath={.metadata.namespace}
```
Alternatively, a namespace can be created using kubectl as well.

Create a Namespace:

```
kubectl create ns dev2
```

Create a Deployment:

```
kubectl -n dev2 apply -f deployment.yaml
```
Get Deployments in the newly created Namespace:

```
kubectl get deployments -n dev2
```

Get Deployments in all Namespaces:

```
kubectl get deployments --all-namespaces
```

# Quota and Limits
Each namespace can be assigned resource quota. Specifying quota allows to restrict how much of cluster resources can be consumed across all resources in a namespace. Resource quota can be defined by a ResourceQuota object. A presence of ResourceQuota object in a namespace ensures that resource quotas are enforced. There can be at most one ResourceQuota object in a namespace. Currently, multiple ResourceQuota objects are allowed. This is filed as kubernetes#55430.

A quota can be specified for compute resources such as CPU and memory, storage resources such as PersistentVolume and PersistentVolumeClaim and number of objects of a given type. A complete list of resources that can be restricted using ResourceQuota are listed at https://kubernetes.io/docs/concepts/policy/resource-quotas/.

## Create ResourceQuota
A ResourceQuota can be created using a configuration file or kubectl.

The following configuration file can be used to create ResourceQuota:


resource-quota.yaml
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
spec:
  hard:
    cpu: "4"
    memory: 6G
    pods: "10"
    replicationcontrollers: "3"
    services: "5"
    configmaps: "5"
```



Create a new ResourceQuota:

```
kubectl apply -f resource-quota.yaml
```

Get the list of ResourceQuota:

```
kubectl get quota
```

Get more details about the ResourceQuota:

```
kubectl describe quota/quota
```

The output shows that three Pods and one Service already exists in the default namespace.

Scale resources with ResourceQuota
Now that the ResourceQuota has been created, let’s see how this impacts the new resources that are created or existing resources that are scaled.

We already have a Deployment nginx-deployment. Let’s scale the number of replicas to exceed the assigned quota and see what happens.

Scale the number of replicas for the Deployment:

```
kubectl scale --replicas=12 deployment/nginx-deployment
```
The command output says that the Deployment is scaled.

Let’s check if all the replicas are available:

```
kubectl get deployment/nginx-deployment -o jsonpath={.status.availableReplicas}
```
It shows only three replicas are available.

More details can be found:

```
kubectl describe deployment nginx-deployment
```


Create resources with ResourceQuota

Let’s create a Pod with the following configuration file:

pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```
You may have to remove a previously running Pod or Deployment before attempting to create this Pod.

```
kubectl apply -f pod.yaml
```


Update the configuration file to:

pod-cpu-memory.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        memory: "100m"
    ports:
    - containerPort: 80
```
There is an explicity memory resource defined here. Now, try to create the pod:

```
kubectl apply -f pod-cpu-memory.yaml
```


Get more details about the Pod:

```
kubectl get pod/nginx-pod -o jsonpath={.spec.containers[].resources}
```

```
kubectl delete quota/quota
```

```
kubectl delete quota/quota2
```
