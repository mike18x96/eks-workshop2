# Deployment
A “desired state”, such as 4 replicas of a pod, can be described in a Deployment object. The Deployment controller in Kubernetes cluster then ensures the desired and the actual state are matching. Deployment ensures the recreation of a pod when the worker node fails or reboots. If a pod dies, then a new pod is started to ensure the desired vs actual matches. It also allows both up- and down-scaling the number of replicas. This is achieved using ReplicaSet. The Deployment manages the ReplicaSets and provides updates to those pods.

## Create a Deployment
The folowing example will create a Deployment with 3 replicas of NGINX base image. Let’s begin with the template:

deployment.yaml
```
apiVersion: apps/v1
kind: Deployment # kubernetes object type
metadata:
  name: nginx-deployment # deployment name
spec:
  replicas: 1 # number of replicas
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx # pod labels
    spec:
      containers:
      - name: nginx # container name
        image: nginx:1.12.1 # nginx image
        imagePullPolicy: IfNotPresent # if exists, will not pull new image
        ports: # container and host port assignments
        - containerPort: 80
        - containerPort: 443
```
This deployment will created instances of NGINX image.

Run the following command to create Deployment:

```
kubectl create -f deployment.yaml
```


To monitor deployment rollout status:

```
kubectl rollout status deployment/nginx-deployment
```

A Deployment creates a ReplicaSet to manage the number of replicas. Let’s take a look at existing deployments and replica set.

Get the deployments:

```
kubectl get deployments
```

Get the replica set for the deployment:

```
kubectl get replicaset
```

Get the list of running pods:

```
kubectl get pods
```

## Scaling a Deployment
Number of replicas for a Deployment can be scaled using the following command:

```
kubectl scale --replicas=3 deployment/nginx-deployment
```
deployment "nginx-deployment" scaled
Verify the deployment:

```
kubectl get deployments
```

Verify the pods in the deployment:

```
kubectl get pods
```

## Update a Deployment
A more general update to Deployment can be made by making edits to the pod spec. In this example, let’s change to the latest nginx image.

First, type the following to open up a text editor:

```
kubectl edit deployment/nginx-deployment
```
Next, change the image from nginx:1.12.1 to nginx:latest.

This should perform a rolling update of the deployment. To track the deployment details such as revision, image version, and ports - type in the following:

```
kubectl describe deployments
```

## Rollback a Deployment
To rollback to a previous version, first check the revision history:

```
kubectl rollout history deployment/nginx-deployment
```

If you only want to rollback to the previous revision, enter the following command:

```
kubectl rollout undo deployment/nginx-deployment
```
deployment "nginx-deployment" rolled back

In our case, the deployment will rollback to use the nginx:1.12.1 image. Check the image name:

```
kubectl describe deployments | grep Image
```

If rolling back to a specific revision then enter:

```
kubectl rollout undo deployment/nginx-deployment --to-revision=<version>
```
Delete a Deployment
Run the following command to delete the Deployment:

```
kubectl delete -f deployment.yaml
```
deployment "nginx-deployment" deleted
