# Daemon Set
Daemon Set ensure that a copy of the pod runs on a selected set of nodes. By default, all nodes in the cluster are selected. A selection critieria may be specified to select a limited number of nodes.

As new nodes are added to the cluster, pods are started on them. As nodes are removed, pods are removed through garbage collection.

## Create a DaemonSet
The following is an example DaemonSet that runs a Prometheus container. Let’s begin with the template:

daemonset.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: prometheus-daemonset
spec:
  selector:
    matchLabels:
      tier: monitoring
      name: prometheus-exporter
  template:
    metadata:
      labels:
        tier: monitoring
        name: prometheus-exporter
    spec:
      containers:
      - name: prometheus
        image: prom/node-exporter
        ports:
        - containerPort: 80
```
Run the following command to create the ReplicaSet and pods:

```
kubectl create -f daemonset.yaml
```


Get basic details about the DaemonSet:

```
kubectl get daemonsets/prometheus-daemonset
```

Get more details about the DaemonSet:

```
kubectl describe daemonset/prometheus-daemonset
```

Get pods in the DaemonSet:

```
kubectl get pods -lname=prometheus-exporter
```

Limit DaemonSets to specific nodes
Verify that the Prometheus pod was successfully deployed to the cluster nodes:

```
kubectl get pods -o wide
```

Rename one of the node labels as follows:

```
kubectl label node ip-172-20-52-200.ec2.internal app=prometheus-node
```

Next, edit the DaemonSet template using the command shown:

```
kubectl edit ds/prometheus-daemonset
```
Change the spec.template.spec to include a nodeSelector that matches the changed label:

      nodeSelector:
        app: prometheus-node
After the update is performed, we have now configured Prometheus to run on a specific node:

```
kubectl get ds/prometheus-daemonset
```

Delete a DaemonSet

```
kubectl delete -f daemonset.yaml
```
