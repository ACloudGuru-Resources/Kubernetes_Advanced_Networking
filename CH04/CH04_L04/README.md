# Services Expanded Lab

### Objectives
1. Explore Headless,External and Cluster IP Services

### Prereq
1. Running kind cluster

### Setup Steps

1. Deploy Kind Cluster
2. Deploy DNS utils images
3. Deploy Services
4. Explore Service Network

### 1. Deploy Kind Cluster 

1. Start kind cluster

```bash
kind create cluster --name service --config kind.yml
```

```bash
± |master {1} S:5 U:4 ?:1 ✗| → kind create cluster --config kind.config --name nodeport
Creating cluster "service" ...
 ✓ Ensuring node image (kindest/node:v1.16.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-service"
You can now use your cluster with:

kubectl cluster-info --context kind-service

Have a nice day! 👋
```
 
### 2. Deploy DNS utils images

dnsutils is used for [kubernetes end to end testing](https://github.com/kubernetes/kubernetes/tree/master/test/images)
 
```bash
kubectl apply -f dnsutils.yml 
```

### 3. Deploy Services 

You can deploy one by one or all together 

```bash
kubectl apply -f service-headless.yml,service.yml,service-external.yml
```

```bash
service/headless-service unchanged
deployment.apps/app created
service/clusterip-service created
service/external-service created
```

ClusterIP Service

```bash
kubectl apply -f service.yml
```
Headless Service

```bash
kubectl apply -f service-headless.yml
```

External Service

```bash
kubectl apply -f external-headless.yml
```

Verify everything deploy successfully 

Resolve Cluster IP service
```bash
 kubectl exec -it dnsutils -- host -v -t a clusterip-service
```

```bash
Trying "clusterip-service.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62956
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;clusterip-service.default.svc.cluster.local. IN        A

;; ANSWER SECTION:
clusterip-service.default.svc.cluster.local. 30 IN A 10.102.42.19

Received 120 bytes from 10.96.0.10#53 in 0 ms
```

Resolve Headless service

```bash
kubectl exec -it dnsutils -- host -v -t a headless-service
```
```bash
Trying "headless-service.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26926
;; flags: qr aa rd; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;headless-service.default.svc.cluster.local. IN A

;; ANSWER SECTION:
headless-service.default.svc.cluster.local. 30 IN A 10.244.3.3
headless-service.default.svc.cluster.local. 30 IN A 10.244.1.2
headless-service.default.svc.cluster.local. 30 IN A 10.244.1.4
headless-service.default.svc.cluster.local. 30 IN A 10.244.2.3
headless-service.default.svc.cluster.local. 30 IN A 10.244.3.2
headless-service.default.svc.cluster.local. 30 IN A 10.244.2.2
headless-service.default.svc.cluster.local. 30 IN A 10.244.1.3

Received 466 bytes from 10.96.0.10#53 in 0 ms
```

Resolve External service

```bash
kubectl exec -it dnsutils -- host -v -t a external-service
```
```bash
Trying "external-service.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17944
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;external-service.default.svc.cluster.local. IN A

;; ANSWER SECTION:
external-service.default.svc.cluster.local. 30 IN CNAME google.com.
google.com.             30      IN      A       172.217.5.14

Received 152 bytes from 10.96.0.10#53 in 15 ms
```


### 4. Explore Services

Let's Debug one of the pods from the Service 

```bash
kubectl get pods -l app=app1
```

Run Pod watcher in a separate terminal, you'll see the Deployment create a new pod since the label changed, which in
 turn will create a new Endpoint 
  
```bash
kubectl get pods --watch
```

```bash
NAME                  READY   STATUS    RESTARTS   AGE
app-ff9cd5f65-76xhg   1/1     Running   0          7s
app-ff9cd5f65-d55vk   1/1     Running   0          7s
app-ff9cd5f65-hlfqd   1/1     Running   0          7s
app-ff9cd5f65-lcwks   1/1     Running   0          7s
app-ff9cd5f65-lvpkq   1/1     Running   0          7s
app-ff9cd5f65-p6rwq   1/1     Running   0          7s
app-ff9cd5f65-qh4qs   1/1     Running   0          7s
dnsutils              1/1     Running   0          14m
app-ff9cd5f65-76xhg   1/1     Running   0          36s
app-ff9cd5f65-76xhg   1/1     Running   0          36s
app-ff9cd5f65-phhfv   0/1     Pending   0          0s
app-ff9cd5f65-phhfv   0/1     Pending   0          0s
app-ff9cd5f65-phhfv   0/1     ContainerCreating   0          0s
app-ff9cd5f65-phhfv   1/1     Running             0          1s
```

Change the Label to remove the App from the Service and Endpoints
```bash
 kubectl label pod app-ff9cd5f65-2n77z app=debug --overwrite=true
```

You'll see 8 pods running now 
```bash
kubectl get pods -o wide --show-labels
```
```bash
NAME                  READY   STATUS    RESTARTS   AGE     LABELS
app-ff9cd5f65-76xhg   1/1     Running   0          4m3s    app=debug,pod-template-hash=ff9cd5f65
app-ff9cd5f65-8wfr2   1/1     Running   0          10s     app=app1,pod-template-hash=ff9cd5f65
app-ff9cd5f65-d55vk   1/1     Running   0          4m3s    app=debug,pod-template-hash=ff9cd5f65
app-ff9cd5f65-hlfqd   1/1     Running   0          4m3s    app=app1,pod-template-hash=ff9cd5f65
app-ff9cd5f65-lcwks   1/1     Running   0          4m3s    app=app1,pod-template-hash=ff9cd5f65
app-ff9cd5f65-lvpkq   1/1     Running   0          4m3s    app=app1,pod-template-hash=ff9cd5f65
app-ff9cd5f65-p6rwq   1/1     Running   0          4m3s    app=app1,pod-template-hash=ff9cd5f65
app-ff9cd5f65-phhfv   1/1     Running   0          3m27s   app=app1,pod-template-hash=ff9cd5f65
app-ff9cd5f65-qh4qs   1/1     Running   0          4m3s    app=app1,pod-template-hash=ff9cd5f65
dnsutils              1/1     Running   0          18m     <none>


```
It will be removed from the Endpoints for the services 
```bash 
kubectl describe endpoints clusterip-service
```

```bash
   Name:         clusterip-service
   Namespace:    default
   Labels:       app=app1
   Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2020-04-25T20:02:12Z
   Subsets:
     Addresses:          10.244.1.2,10.244.1.3,10.244.2.2,10.244.2.3,10.244.2.4,10.244.3.2,10.244.3.3
     NotReadyAddresses:  <none>
     Ports:
       Name     Port  Protocol
       ----     ----  --------
       <unset>  8080  TCP
   
   Events:  <none>
```

Let's Explore the Effects of DNS resolutions on google.com using an External Service

```bash
 kubectl exec -it dnsutils -- host -v -t a google.com 8.8.8.8
 Trying "google.com.default.svc.cluster.local"
 Trying "google.com.svc.cluster.local"
 Trying "google.com.cluster.local"
 Trying "google.com"
 Using domain server:
 Name: 8.8.8.8
 Address: 8.8.8.8#53
 Aliases: 
 
 ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40188
 ;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 0
 
 ;; QUESTION SECTION:
 ;google.com.                    IN      A
 
 ;; ANSWER SECTION:
 google.com.             281     IN      A       64.233.185.139
 google.com.             281     IN      A       64.233.185.138
 google.com.             281     IN      A       64.233.185.101
 google.com.             281     IN      A       64.233.185.100
 google.com.             281     IN      A       64.233.185.113
 google.com.             281     IN      A       64.233.185.102
 
 Received 124 bytes from 8.8.8.8#53 in 15 ms
```

```bash
kubectl exec -it dnsutils -- host -v -t a google.com
```
```bash
Trying "google.com.default.svc.cluster.local"
Trying "google.com.svc.cluster.local"
Trying "google.com.cluster.local"
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11603
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             30      IN      A       172.217.5.14

Received 54 bytes from 10.96.0.10#53 in 2 ms
```

```bash
kubectl exec -it dnsutils -- host -v -t a external-service
```

```
Trying "external-service.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11610
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;external-service.default.svc.cluster.local. IN A

;; ANSWER SECTION:
external-service.default.svc.cluster.local. 20 IN CNAME google.com.
google.com.             20      IN      A       172.217.5.14

Received 152 bytes from 10.96.0.10#53 in 0 ms
```




