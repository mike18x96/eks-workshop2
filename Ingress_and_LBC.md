# AWS LOAD BALANCER CONTROLLER

The AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster. The controller provisions the following resources:

An AWS Application Load Balancer (ALB) when you create a Kubernetes Ingress.

An AWS Network Load Balancer (NLB) when you create a Kubernetes service of type LoadBalancer. In the past, the Kubernetes network load balancer was used for instance targets, but the AWS Load balancer Controller was used for IP targets. With the AWS Load Balancer Controller version 2.3.0 or later, you can create NLBs using either target type. For more information about NLB target types, see Target type in the User Guide for Network Load Balancers.

The AWS Load Balancer Controller was formerly named the AWS ALB Ingress Controller. It's an open-source project managed on GitHub.

https://github.com/kubernetes-sigs/aws-load-balancer-controller

## Installing LB controller

### Add OIDC provider to the cluster
Replace **dev-cluster** with you EKS cluster
```
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=dev-cluster
```
### Create IAM policy

#### Download policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.7/docs/install/iam_policy.json
```
#### Create policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

#### Create Role
Replace **dev-cluster** with you EKS cluster name and **111122223333** with your AWS account id
```
eksctl create iamserviceaccount \
  --cluster=dev-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
  ```
  
  #### Install Cert Manager
  ```
  kubectl apply \
    --validate=false \
    -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml

  ```
  
  #### Install LB controller
  ```
  curl -Lo v2_4_7_full.yaml https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.7/v2_4_7_full.yaml
  ```
  
  #### Remove Service account from the file
  ```
  sed -i.bak -e '561,569d' ./v2_4_7_full.yaml
  ```
  
  #### Replace cluster name
  Change **dev-cluster** to you cluster name
  ```
  sed -i.bak -e 's|your-cluster-name|dev-cluster|' ./v2_4_7_full.yaml
  ```
  
  #### For Fargate and EC2 with restricted access to IMDS(OPTIONAL) add following parameters
  Set correct cluset-name, aws-vpc-id and aws-region.
  ```
  ...
spec:
      containers:
        - args:
            - --cluster-name=dev-cluster
            - --ingress-class=alb
            - --aws-vpc-id=vpc-xxxxxxxx
            - --aws-region=us-west-2
            
            
...
  ```
  
  ### Apply manifest
  ```
  kubectl apply -f v2_4_7_full.yaml
  ```
  
  ### Download IngressClass and IngressClassParams
  ```
  curl -Lo v2_4_7_ingclass.yaml https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.7/v2_4_7_ingclass.yaml
  ```
  
  ### Apply manifest
  ```
  kubectl apply -f v2_4_7_ingclass.yaml
  ```
  ## Verify if ALB is running
  ```
  kubectl get deployment -n kube-system aws-load-balancer-controller
  ```
  
  # INGRESS with AWS LOAD BALANCER CONTROLLER
  Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

Details: https://kubernetes.io/docs/concepts/services-networking/ingress/

   ## Example manifest stub
   Now you goal is to create you own Docker image, build and push to ECS, edit this stub and test you own application!
   Replace ports to match you application, ecr image and repo and names
   ```
   ---
apiVersion: v1
kind: Namespace
metadata:
  name: webapp
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: webapp
  name: deployment-webapp
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: app-webapp
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-webapp
    spec:
      containers:
      - image: <account_id>.dkr.ecr.us-west-2.amazonaws.com/<repo1>:latest
        imagePullPolicy: Always
        name: app-webapp
        ports:
        - containerPort: 8001
---
apiVersion: v1
kind: Service
metadata:
  namespace: webapp
  name: service-webapp
spec:
  ports:
    - port: 8001
      targetPort: 8001
      protocol: TCP
  type: NodePort
  selector:
    app.kubernetes.io/name: app-webapp
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  namespace: webapp
  name: ingress-webapp
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: service-webapp
              servicePort: 8001
   ```
