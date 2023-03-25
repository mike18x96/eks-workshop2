# PODS
A Pod is the smallest deployable unit that can be created, scheduled, and managed. It’s a logical collection of containers that belong to an application. Pods are created in a namespace. All containers in a pod share the namespace, volumes and networking stack. This allows containers in the pod to “find” each other and communicate using localhost.

## Create a Pod using pod.yaml
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

Create the pod as shown below:
```
kubectl apply -f pod.yaml
```

Get the list of pods
```
kubectl get pods
```

Run port forward 
```
kubectl -n default port-forward $(kubectl -n default get pod -l name=nginx-pod -o jsonpath='{.items[0].metadata.name}') 8080:80
```

```
curl localhost:8080
```

You should see nginx webpage


## Memory and CPU resource request
A Container in a Pod can be assigned memory and CPU request and limit. Request is the minimum amount of memory/CPU that Kubernetes will give to the container. Limit is the maximum amount of memory/CPU that a container will be allowed to use. The memory/CPU request/limit for the Pod is the sum of the memory/CPU requests/limits for all the Containers in the Pod. Request defaults to limit if not specified. Default value of the limit is the node capacity.

A Pod can be scheduled on a node if the Pod’s memory and CPU request can be met. Memory and CPU limits are not taken into consideration for scheduling.

Pod can continue to operate on the node if Containers in the Pod does not exceed the memory request. If Containers in the Pod exceeds the memory request then they become target of eviction whenever the node runs out of memory. If Containers in the Pod exceeds the memory limit then they are terminated. If the Pod can be restarted, then kubelet will restart it, just like any other type of runtime failure. A Container might or might not be allowed to exceed its CPU limit for extended periods of time. However, it will not be killed for excessive usage.

Memory and CPU request/limit can be specified using the following:


Memory request
```
spec.containers[].resources.requests.memory
```
Memory limit
```
spec.containers[].resources.limits.memory
```
CPU request
```
spec.containers[].resources.requests.cpu
```
CPU limit
```
spec.containers[].resources.limits.cpu
```
Memory resources are requested in bytes. You can specify them in integer or decimals with one of the suffixes E, P, T, G, M, K. It can also be expressed with power-of-two equivalents Ei, Pi, Ti, Gi, Mi, Ki.

CPU can be requested in cpu units. 1 cpu unit is equivalent 1 AWS vCPU. It can also be requested in fractional units, such as 0.5 or in millicpu such as 500m.

Default memory and CPU
By default, a container in a pod is not allocated any requests or limits. This can be verified using the previously started pod:
```
kubectl get pod/nginx-pod -oyaml
```

Assign memory and CPU
Let’s assign a memory request and limit to a Pod using the configuration file shown:

pod-resources.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod2
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits:
        memory: "200Mi"
        cpu: 2
      requests:
        memory: "100Mi"
        cpu: 1
    ports:
    - containerPort: 80
```
The only change in this configuration file is the addition of spec.containers[].resources section. The limits are specified in the limits section and the requests are specified in the requests section.

Create the pod:
```
kubectl apply -f pod-resources.yaml
```

Get more details about the requests and limits:
```
kubectl get pod/nginx-pod2 -o jsonpath={.spec.containers[].resources}
map[limits:map[memory:200Mi cpu:2] requests:map[cpu:1 memory:100Mi]]
```
Delete the pod
```
kubectl delete pod nginx-pod2
```

NGINX container requires fairly low memory and CPU. And so these request and limit numbers would work well, and the pod is started correctly. Now, let’s try to start a WildFly pod using similar numbers. The configuration file for the same is shown:

pod-resources1.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: wildfly-pod
  labels:
    name: wildfly-pod
spec:
  containers:
  - name: wildfly
    image: jboss/wildfly:11.0.0.Final
    resources:
      limits:
        memory: "120Mi"
        cpu: 2
      requests:
        memory: "100Mi"
        cpu: 1
    ports:
    - containerPort: 8080
```
The max amount of memory allocated for the WildFly container in this pod is restricted to 120MB. Let’s create this Pod:

```
kubectl apply -f pod-resources1.yaml
```
pod "wildfly-pod" created
Watch the status of the Pod:

```
kubectl get pods -w
NAME          READY     STATUS              RESTARTS   AGE
wildfly-pod   0/1       ContainerCreating   0          5s
wildfly-pod   1/1       Running   0         26s
wildfly-pod   0/1       OOMKilled   0         29s
wildfly-pod   1/1       Running   1         31s

```
OOMKilled shows that the container was terminated because it ran out of memory.

To correct this, we’ll need to re-create the pod with higher memory limits.

Although it may be instinctive to simply adjust the memory limit in the existing pod definition and re-apply it, Kubernetes does not currently support changing resource limits on running pods, so we’ll need to first delete the existing pod, then recreate it.

In pod-resources2.yaml, confirm that the value of spec.containers[].resources.limits.memory is 300Mi. Delete the existing Pod, and create a new one:

```
kubectl delete -f pod-resources1.yaml
```
pod "wildfly-pod" deleted
```
kubectl apply -f pod-resources2.yaml
```
pod "wildfly-pod" created
```
kubectl get -w pod/wildfly-pod
```
NAME          READY     STATUS              RESTARTS   AGE
wildfly-pod   0/1       ContainerCreating   0          3s
wildfly-pod   1/1       Running   0         25s
Now, the Pod successfully starts.

Get more details about the resources allocated to the Pod:

```
kubectl get pod/wildfly-pod -o jsonpath={.spec.containers[].resources}
map[limits:map[cpu:2 memory:300Mi] requests:map[cpu:1 memory:100Mi]]
```
## Quality of service
Kubernetes opportunistically scavenges the difference between request and limit if they are not used by the Containers. This allows Kubernetes to oversubscribe nodes, which increases utilization, while at the same time maintaining resource guarantees for the containers that need guarantees.

Kubernetes assigns one of the QoS classes to the Pod:

**Guaranteed**

**Burstable**

**BestEffort**

**QoS **class is used by Kubernetes for scheduling and evicting Pods.

When every Container in a Pod is given a memory and CPU limit, and optionally non-zero request, and they exactly match, then a Pod is scheduled with Guaranteed QoS. This is the highest priority.

A Pod is given Burstable QoS class if the Pod does not meet the Guaranteed QoS and at least one Container has a memory or CPU request. This is intermediate priority.

When no memory and CPU request or limit is assigned to any Container in the Pod, then a Pod is scheduled with BestEffort QoS. This the lowest and the default priority.

Pods that need to stay up can request Guaranteed QoS. Pods with less stringent requirement can use a weaker or no QoS.

Guaranteed
Here is an example of Pod with Guaranteed QoS:


pod-guaranteed.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-guaranteed
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits:
        memory: "200Mi"
        cpu: 1
    ports:
    - containerPort: 80
```
Note that no request values are specified here, and will default to limit.

Create this Pod:

```
kubectl apply -f pod-guaranteed.yaml
```
pod "nginx-pod-guaranteed" created
Check the resources:

```
kubectl get pod/nginx-pod-guaranteed -o jsonpath={.spec.containers[].resources}
map[limits:map[cpu:1 memory:200Mi] requests:map[cpu:1 memory:200Mi]]
```
Check the QoS:

```
kubectl get pod/nginx-pod-guaranteed -o jsonpath={.status.qosClass}
```
Guaranteed
Another Pod with explicit value for limit and request is shown:


pod-guaranteed2.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-guaranteed2
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits:
        memory: "200Mi"
        cpu: 1
      requests:
        memory: "200Mi"
        cpu: 1
    ports:
    - containerPort: 80
```
Create this Pod:

```
kubectl apply -f pod-guaranteed2.yaml
```
pod "nginx-pod-guaranteed2" created
Check the resources:

```
kubectl get pod/nginx-pod-guaranteed2 -o jsonpath={.spec.containers[].resources}
map[limits:map[cpu:1 memory:200Mi] requests:map[cpu:1 memory:200Mi]]
```
Check the QoS:

```
kubectl get pod/nginx-pod-guaranteed2 -o jsonpath={.status.qosClass}
```
Guaranteed
Burstable
Here is an example of Pod with Burstable QoS:

pod-burstable.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-burstable
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits:
        memory: "200Mi"
        cpu: 1
      requests:
        memory: "100Mi"
        cpu: 1
    ports:
    - containerPort: 80
```
Note that both request and limit values are specified here.

Create this Pod:

```
kubectl apply -f pod-burstable.yaml
```

Check the resources:

```
kubectl get pod/nginx-pod-burstable -o jsonpath={.spec.containers[].resources}
map[limits:map[cpu:1 memory:200Mi] requests:map[cpu:1 memory:100Mi]]
```
Check the QoS:
```
kubectl get pod/nginx-pod-burstable -o jsonpath={.status.qosClass}
```
Burstable
BestEffort
Check the resources:

```
kubectl get pod/nginx-pod -o jsonpath={.spec.containers[].resources}
map[requests:map[cpu:100m]]
```
Check the QoS:

```
kubectl get pod/nginx-pod -o jsonpath={.status.qosClass}
```
Burstable
This should be BestEffort and filed as kubernetes#55278.

Delete a Pod
Get all the Pods that are running:

kubectl get pods
NAME                    READY     STATUS    RESTARTS   AGE
nginx-pod               1/1       Running   0          6m
nginx-pod-burstable     1/1       Running   0          9m
nginx-pod-guaranteed    1/1       Running   0          23m
nginx-pod-guaranteed2   1/1       Running   0          12m
nginx-pod2              1/1       Running   0          6m
wildfly-pod             1/1       Running   0          6m
Delete the Pods as shown below:

```
kubectl delete $(kubectl get pods -o=name)
```
