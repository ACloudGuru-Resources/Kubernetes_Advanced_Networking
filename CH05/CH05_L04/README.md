### Objectives
1. Deploy EKS Cluster
2. Deploy ALB Ingress Controller 
3. Deploy Test Application 
4. Verify

### Prereq
1. Deploy an EKS Cluster 

### Setup Steps

### 1. Deploy EKS Cluster
```bash
eksctl create cluster -N 3 --name alb-ingress-lab --alb-ingress-access
```

```bash
[ℹ]  eksctl version 0.12.0
[ℹ]  using region us-west-2
[ℹ]  setting availability zones to [us-west-2c us-west-2d us-west-2b]
[ℹ]  subnets for us-west-2c - public:192.168.0.0/19 private:192.168.96.0/19
[ℹ]  subnets for us-west-2d - public:192.168.32.0/19 private:192.168.128.0/19
[ℹ]  subnets for us-west-2b - public:192.168.64.0/19 private:192.168.160.0/19
[ℹ]  nodegroup "ng-e7e094a6" will use "ami-0c13bb9cbfd007e56" [AmazonLinux2/1.14]
[ℹ]  using Kubernetes version 1.14
[ℹ]  creating EKS cluster "attractive-sculpture-1587846638" in "us-west-2" region with un-managed nodes
[ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial nodegroup
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=attractive-sculpture-1587846638'
[ℹ]  CloudWatch logging will not be enabled for cluster "attractive-sculpture-1587846638" in "us-west-2"
[ℹ]  you can enable it with 'eksctl utils update-cluster-logging --region=us-west-2 --cluster=attractive-sculpture-1587846638'
[ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "attractive-sculpture-1587846638" in "us-west-2"
[ℹ]  2 sequential tasks: { create cluster control plane "attractive-sculpture-1587846638", create nodegroup "ng-e7e094a6" }
[ℹ]  building cluster stack "eksctl-attractive-sculpture-1587846638-cluster"
[ℹ]  deploying stack "eksctl-attractive-sculpture-1587846638-cluster"
[ℹ]  building nodegroup stack "eksctl-attractive-sculpture-1587846638-nodegroup-ng-e7e094a6"
[ℹ]  --nodes-min=3 was set automatically for nodegroup ng-e7e094a6
[ℹ]  --nodes-max=3 was set automatically for nodegroup ng-e7e094a6
[ℹ]  deploying stack "eksctl-attractive-sculpture-1587846638-nodegroup-ng-e7e094a6"
[✔]  all EKS cluster resources for "attractive-sculpture-1587846638" have been created
[✔]  saved kubeconfig as "/Users/strongjz/.kube/config"
[ℹ]  adding identity "arn:aws:iam::725406136353:role/eksctl-attractive-sculpture-15878-NodeInstanceRole-8VILH6IFQH5S" to auth ConfigMap
[ℹ]  nodegroup "ng-e7e094a6" has 0 node(s)
[ℹ]  waiting for at least 3 node(s) to become ready in "ng-e7e094a6"
[ℹ]  nodegroup "ng-e7e094a6" has 3 node(s)
[ℹ]  node "ip-192-168-13-117.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-36-240.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-81-157.us-west-2.compute.internal" is ready
[ℹ]  kubectl command should work with "/Users/strongjz/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "attractive-sculpture-1587846638" in "us-west-2" region is ready
```

### 2. Deploy ALB Ingress Controller 

2.1 Deploy rbac role for ALB Ingress Controller

```bash
kubectl apply -f rbac-role.yml
```
```bash
```

2.2 Deploy ALB Ingress Controller
```bash
kubectl apply -f alb-ingress-contrller.yml
```
```bash
```

Verify the ALB Deployment

```bash
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "alb-ingress[a-zA-Z0-9-]+")

```

### 3. Deploy Test Application 

DNS Utils
```bash
kubectl apply -f dnsutils.yml
```
```bash

```

Database Deployment
```bash
kubectl apply -f database.yml
```
```bash

```

Application Deployment
```bash
kubectl apply -f app.yml
```
```bash

```

Verify Services are reachable inside the cluster
```bash 
kc exec -it dnsutils -- wget -qO- ping-service/host
{"message":"NODE: ip-192-168-13-117.us-west-2.compute.internal, POD IP:192.168.30.127"}

kc exec -it dnsutils -- wget -qO- ping-service/ping
{"message":"pong"}

kc exec -it dnsutils -- wget -qO- data-service/data
{"message":"Database Connected"}

kc exec -it dnsutils -- wget -qO- data-service/host
{"message":"NODE: ip-192-168-81-157.us-west-2.compute.internal, POD IP:192.168.83.48"}

kc exec -it dnsutils -- wget -qO- admin-service:8090/admin
{"message":"Admin Sections"}

kc exec -it dnsutils -- wget -qO- admin-service/host
{"message":"NODE: ip-192-168-36-240.us-west-2.compute.internal, POD IP:192.168.42.253"}
```

Deploy ALB Ingres Rule for Application 
```bash
kubectl apply -f alb-ingress-rule.yml
```

```bash

```


### 4. Verify

Verify the alb-ingress-controller creates the resources
```bash
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o 'alb-ingress[a-zA-Z0-9-]+') | grep 'ping\/ping'
```

Check the events of the ingress to see what has occur.

```bash
kubectl describe ing app
```
```bash

```

Verify Applications is reachable over ALB 

```bash
```

Clean Up 

```bash
kubectl delete -f app-ingress-rule.yml,alb-ingress-controll.yml

eksctl delete cluster --name alb-ingress-lab
```