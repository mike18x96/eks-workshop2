# Service
A pod is ephemeral. Each pod is assigned a unique IP address. If a pod that belongs to a replication controller dies, then it is recreated and may be given a different IP address. Further, additional pods may be created using Deployment or Replica Set. This makes it difficult for an application server, such as WildFly, to access a database, such as MySQL, using its IP address.

A Service is an abstraction that defines a logical set of pods and a policy by which to access them. The IP address assigned to a service does not change over time, and thus can be relied upon by other pods. Typically, the pods belonging to a service are defined by a label selector. This is similar mechanism to how pods belong to a replica set.

This abstraction of selecting pods using labels enables a loose coupling. The number of pods in the deployment may scale up or down but the application server can continue to access the database using the service.

A Kubernetes service defines a logical set of pods and enables them to be accessed through microservices.

Create a Deployment for Service
Pods belong to a service by using a loosely-coupled model where labels are attached to a pod and a service picks the pods by using those labels.

Let’s create a Deployment first that will create 3 replicas of a pod:

echo-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo-pod
  template:
    metadata:
      labels:
        app: echo-pod
    spec:
      containers:
      - name: echoheaders
        image: k8s.gcr.io/echoserver:1.10
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
```
This example creates an echo app that responds with HTTP headers from an Elastic Load Balancer.

Type the following to create the deployment:

```
kubectl apply -f echo-deployment.yaml
```
Use the kubectl describe deployment command to confirm echo-app has been deployed:

```
kubectl describe deployment
```

Get the list of pods:

```
kubectl get pods
```

Check the label for a pod:

```
kubectl describe pods/echo-deployment-3396249933-8slzp | grep Label
```

Each pod in this deployment has app=echo-pod label attached to it.

## Create a Service
In the following example, we create a service echo-service:

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: echo-service
spec:
  selector:
    app: echo-pod
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```
The set of pods targeted by the service are determined by the label app: echo-pod attached to them. It also defines an inbound port 80 to the target port of 8080 on the container.

Kubernetes supports both TCP and UDP protocols.

## Publish a Service
A service can be published to an external IP using the type attribute. This attribute can take one of the following values:

**ClusterIP**: Service exposed on an IP address inside the cluster. This is the default behavior.
**NodePort**: Service exposed on each Node’s IP address at a defined port.
**LoadBalancer**: If deployed in the cloud, exposed externally using a cloud-specific load balancer.
**ExternalName**: Service is attached to the externalName field. It is mapped to a CNAME with the value.

Let’s publish the service load balancer and expose your services, add a type field of LoadBalancer.

This template will expose echo-app service on an Elastic Load Balancer (ELB):

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: echo-service
spec:
  selector:
    app: echo-pod
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```
Run the following command to create the service:

```
kubectl apply -f service.yaml
```
Get more details about the service:

```
kubectl get service
```
EKS create Classic internet-facing load balancer

```
kubectl describe service echo-service
```

```
curl SOME_UNIQUE_ID.us-west-2.elb.amazonaws.com
```

Note the client_address value shown in the output. This is the IP address of the pod serving the request. Multiple invocations of this command will show different values for this attribute.

Now, the number of pods in the deployment can be scaled up and down. Or the pods may terminate and restart on a different host. But the service will still be able to target those pods because of the labels attached to the pod and used by the service.

## Delete a Service
Run the following command to delete the Service:

```
kubectl delete -f service.yaml
```
The backend Deployment needs to be explicitly deleted as well:

```
kubectl delete -f echo-deployment.yaml
```
