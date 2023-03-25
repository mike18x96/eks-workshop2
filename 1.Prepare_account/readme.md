# Setup for EKS with Kubernetes 1.25

# AWS CLI

Install: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

```
$ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
$ sudo installer -pkg AWSCLIV2.pkg -target /
```

Configure: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html

```
aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

```
aws sts get-caller-identity
```


# EKSCTL

Install: https://eksctl.io/introduction/#installation

```
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
```


# KUBECTL 1.25

Install: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.6/2023-01-30/bin/darwin/amd64/kubectl
chmod +x ./kubectl


kubectl version --client
```

# AWS IAM AUTHENTICATOR(not neede if  aws cli is 1.16.156 or later)

Install: https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html

```
brew install aws-iam-authenticator
aws-iam-authenticator help
```


