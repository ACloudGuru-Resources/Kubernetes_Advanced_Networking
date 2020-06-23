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
[ℹ]  adding identity "arn:aws:iam::363534682973:role/eksctl-attractive-sculpture-15878-NodeInstanceRole-8VILH6IFQH5S" to auth ConfigMap
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
clusterrole.rbac.authorization.k8s.io/alb-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/alb-ingress-controller created
serviceaccount/alb-ingress-controller created
```

2.2 Deploy ALB Ingress Controller
```bash
kubectl apply -f alb-ingress-controller.yml
```
```bash
deployment.apps/alb-ingress-controller created
```

Verify the ALB Deployment

```bash
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "alb-ingress[a-zA-Z0-9-]+")
```
```bash
-------------------------------------------------------------------------------
AWS ALB Ingress controller
  Release:    v1.1.6
  Build:      git-95ee2ac8
  Repository: https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git
-------------------------------------------------------------------------------

W0428 23:08:56.332362       1 client_config.go:549] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0428 23:08:59.488628       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null}}}
I0428 23:08:59.488828       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null},"spec":{},"status":{"loadBalancer":{}}}}
I0428 23:08:59.488881       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"=
I0428 23:08:59.489541       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null},"spec":{},"status":{"loadBalancer":{}}}}
I0428 23:08:59.489584       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"=
I0428 23:08:59.489806       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null}}}
I0428 23:08:59.490381       1 controller.go:121] kubebuilder/controller "level"=0 "msg"="Starting EventSource"  "controller"="alb-ingress-controller" "source"={"Type":{"metadata":{"creationTimestamp":null},"spec":{},"status":{"daemonEndpoints":{"kubeletEndpoint":{"Port":0}},"nodeInfo":{"machineID":"","systemUUID":"","bootID":"","kernelVersion":"","osImage":"","containerRuntimeVersion":"","kubeletVersion":"","kubeProxyVersion":"","operatingSystem":"","architecture":""}}}}
I0428 23:08:59.492085       1 leaderelection.go:205] attempting to acquire leader lease  kube-system/ingress-controller-leader-alb...
I0428 23:08:59.503650       1 leaderelection.go:214] successfully acquired lease kube-system/ingress-controller-leader-alb
I0428 23:08:59.503746       1 recorder.go:53] kubebuilder/manager/events "level"=1 "msg"="Normal"  "message"="alb-ingress-controller-77cc955b69-q999n_3c5df4cc-89a5-11ea-b22d-8a873867c271 became leader" "object"={"kind":"ConfigMap","namespace":"kube-system","name":"ingress-controller-leader-alb","uid":"3e3c4aa2-89a5-11ea-927a-0ad365b8a3c8","apiVersion":"v1","resourceVersion":"1384"} "reason"="LeaderElection"
I0428 23:08:59.604221       1 controller.go:134] kubebuilder/controller "level"=0 "msg"="Starting Controller"  "controller"="alb-ingress-controller"
I0428 23:08:59.704375       1 controller.go:154] kubebuilder/controller "level"=0 "msg"="Starting workers"  "controller"="alb-ingress-controller" "worker count"=1

```
### 3. Deploy Test Application 

3.1 DNS Utils
```bash
kubectl apply -f dnsutils.yml
```
```bash
pod/dnsutils created

kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
dnsutils   1/1     Running   0          12s
```

3.2 Database Deployment
```bash
kubectl apply -f database.yml
```
```bash
service/postgres created
configmap/postgres-config created
statefulset.apps/postgres created
```

3.3 Application Deployment
```bash
kubectl apply -f app.yml
```

```bash
deployment.apps/ping created
deployment.apps/data created
deployment.apps/admin created
```

3.4 Deploy Application Services
```bash
 kubectl apply -f app-nodeport-service.yml
```
```bash
service/ping-service created
service/data-service created
service/admin-service created
```

3.5 Verify Services are reachable inside the cluster
```bash 
± |master {1} U:1 ✗| → kc exec -it dnsutils -- wget -qO- ping-service/host
{"message":"NODE: ip-192-168-37-167.us-west-2.compute.internal, POD IP:192.168.61.137"}

kubectl exec -it dnsutils -- wget -qO- ping-service/host
{"message":"NODE: ip-192-168-37-167.us-west-2.compute.internal, POD IP:192.168.61.137"}

kubectl exec -it dnsutils -- wget -qO- data-service/host
{"message":"NODE: ip-192-168-22-1.us-west-2.compute.internal, POD IP:192.168.29.180"}

kubectl exec -it dnsutils -- wget -qO- admin-service/host
{"message":"NODE: ip-192-168-37-167.us-west-2.compute.internal, POD IP:192.168.42.187"}

kubectl exec -it dnsutils -- wget -qO- admin-service/data
{"message":"Database Connected"}

kubectl exec -it dnsutils -- wget -qO- ping-service/data
{"message":"Database Connected"}

kubectl exec -it dnsutils -- wget -qO- data-service/data
{"message":"Database Connected"}
```

Deploy ALB Ingres Rule for Application 
```bash
kubectl apply -f alb-ingress-rule.yml
```

```bash
ingress.extensions/ping created
ingress.extensions/data created
ingress.extensions/admin created
```

### 4. Verify

```bash
kubectl get ing
```
```bash
NAME    HOSTS   ADDRESS                                                             PORTS   AGE
admin   *       764a4912-default-admin-89fb-842744177.us-west-2.elb.amazonaws.com   80      37s
data    *       764a4912-default-data-2758-1454146277.us-west-2.elb.amazonaws.com   80      94s
ping    *       764a4912-default-ping-2c80-520816113.us-west-2.elb.amazonaws.com    80      4m46s
```

Verify the alb-ingress-controller creates the resources
```bash
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o 'alb-ingress[a-zA-Z0-9-]+') | grep 'ping'
```

Check the events of the ingress to see what has occured.

```bash
kubectl describe ing ping
```
```bash
kubectl describe ing
Name:             admin
Namespace:        default
Address:          764a4912-default-admin-89fb-842744177.us-west-2.elb.amazonaws.com
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /admin   admin-service:80 (192.168.42.187:8080)
Annotations:  alb.ingress.kubernetes.io/scheme: internet-facing
              kubernetes.io/ingress.class: alb
Events:
  Type    Reason  Age    From                    Message
  ----    ------  ----   ----                    -------
  Normal  CREATE  5m39s  alb-ingress-controller  LoadBalancer 764a4912-default-admin-89fb created, ARN: arn:aws:elasticloadbalancing:us-west-2:363534682973:loadbalancer/app/764a4912-default-admin-89fb/ea8f3571283cce18
  Normal  CREATE  5m38s  alb-ingress-controller  rule 1 created with conditions [{    Field: "path-pattern",    PathPatternConfig: {      Values: ["/admin"]    }  }]


Name:             data
Namespace:        default
Address:          764a4912-default-data-2758-1454146277.us-west-2.elb.amazonaws.com
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /data   data-service:80 (192.168.29.180:8080)
Annotations:  alb.ingress.kubernetes.io/scheme: internet-facing
              kubernetes.io/ingress.class: alb
Events:
  Type    Reason  Age    From                    Message
  ----    ------  ----   ----                    -------
  Normal  CREATE  6m37s  alb-ingress-controller  LoadBalancer 764a4912-default-data-2758 created, ARN: arn:aws:elasticloadbalancing:us-west-2:363534682973:loadbalancer/app/764a4912-default-data-2758/45a8da8dc2b9447e
  Normal  CREATE  6m36s  alb-ingress-controller  rule 1 created with conditions [{    Field: "path-pattern",    PathPatternConfig: {      Values: ["/data"]    }  }]


Name:             ping
Namespace:        default
Address:          764a4912-default-ping-2c80-520816113.us-west-2.elb.amazonaws.com
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /ping   ping-service:80 (192.168.61.137:8080)
Annotations:  alb.ingress.kubernetes.io/scheme: internet-facing
              kubernetes.io/ingress.class: alb
Events:
  Type    Reason  Age    From                    Message
  ----    ------  ----   ----                    -------
  Normal  CREATE  9m49s  alb-ingress-controller  LoadBalancer 764a4912-default-ping-2c80 created, ARN: arn:aws:elasticloadbalancing:us-west-2:363534682973:loadbalancer/app/764a4912-default-ping-2c80/d849c71b26b29b06
  Normal  CREATE  9m48s  alb-ingress-controller  rule 1 created with conditions [{    Field: "path-pattern",    PathPatternConfig: {      Values: ["/ping"]    }  }]
```

```bash

```

You can view the ALB Ingress logs as well 

Get the Pods for the ALB controller
```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=alb-ingress-controller
```
```bash
NAME                                      READY   STATUS    RESTARTS   AGE
alb-ingress-controller-77cc955b69-q999n   1/1     Running   0          60m
```

Verify Applications is reachable over ALB 


```bash
wget -qO- 764a4912-default-app-c21c-1446509588.us-west-2.elb.amazonaws.com/data
```
```bash
{"message":"Database Connected"}
```

```
 wget -qO- 764a4912-default-app-c21c-1446509588.us-west-2.elb.amazonaws.com/admin
```
```
{"message":"Admin Sections"}
```

```
wget -qO- 764a4912-default-app-c21c-1446509588.us-west-2.elb.amazonaws.com/ping
```
```
{"message":"pong"}
```

Clean Up 

```bash
kubectl delete -f alb-ingress-controller.yml,alb-ingress-rule.yml,app-nodeport-service.yml
```
```
deployment.apps "alb-ingress-controller" deleted
ingress.extensions "app" deleted
service "ping-service" deleted
service "data-service" deleted
service "admin-service" deleted
```

Delete the Cluster
```
eksctl delete cluster --name alb-ingress-lab
```