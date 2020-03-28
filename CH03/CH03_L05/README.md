# Securing the Network with Network Policies

### Objectives
1. Secure application and database traffic

### Prereq
1. Cluster running with Cilium installed

### Setup Steps
1. Deploy EKS Cluster 

2. Deploy Pods and Services for testing

3. Test Connectivity 

4. Deploy Network Policies 

### 1. Deploy EKS Cluster 

If the cluster from Lecture 4 is not running, walk through the Deployment from [Ch03_L04 - Setting up an overlay network](CH03/CH03_L04)
 
### 2. Deploy Pods and Services for testing

2.1 Deploy Application Pods 

```bash
kubectl apply -f app.yml 
```

2.2 Deploy Database pods

```bash
kubectl apply -f database.yml 
```

Deploy Dns Utils pod 

```bash
kubectl apply -f dnsutils.yml
```

### 3. Test Connectivity

Layer 3 Test

Get Pod IP's 
```bash
± |master {1} U:3 ?:7 ✗| → kubectl get pods -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP           NODE                     NOMINATED NODE   READINESS GATES
app-9947fd97f-9mv8h   1/1     Running   0          16m   10.244.0.5   test-net-control-plane   <none>           <none>
app-9947fd97f-bzt6j   1/1     Running   0          16m   10.244.0.4   test-net-control-plane   <none>           <none>
dnsutils              1/1     Running   0          16m   10.244.0.6   test-net-control-plane   <none>           <none>
mysql-0               1/1     Running   0          12m   10.244.0.7   test-net-control-plane   <none>           <none>
mysql-1               1/1     Running   0          11m   10.244.0.8   test-net-control-plane   <none>           <none>
mysql-2               1/1     Running   0          11m   10.244.0.9   test-net-control-plane   <none>           <none>
```

Using Netcat let's scan the IP

```bash
kubectl exec -it dnsutils -- nc -z -vv 10.244.0.5 8080
10.244.0.5 (10.244.0.5:8080) open
sent 0, rcvd 0
```

Ping test
```bash
kubectl exec -it dnsutils -- ping 10.244.0.5
```

Test connectivity to app port 

```bash
kubectl exec -it dnsutils -- nc -z -v app 8080
```

Test connectivity to app api pong 

```bash
kubectl exec -it dnsutils -- wget -qO- app:8080/ping 
```

Test connectivity to app api secret 

```bash
kubectl exec -it dnsutils -- wget -qO- app:8080/secret 
```

Test connectivity to app api to database

```bash
kubectl exec -it dnsutils -- wget -qO- app:8080/data 
```


### 3. Deploy Network Policies

Deploy cilium Layer 3 policy

```bash
kubectl apply -f layer_3_netpol.yml
```

Test connectivity again 
```bash
kubectl exec -it dnsutils -- nc -z -vv 10.244.0.5 8080
```

Deploy cilium Layer 4 policy

```bash
kubectl apply -f layer_4_netpol.yml
```

Test connectivity again 
```bash
kubectl exec -it dnsutils -- nc -z -vv 10.244.0.5 8080
```

Deploy cilium Layer 7 policy

```bash
kubectl apply -f layer_7_netpol.yml
```

For this test we need to expose the Application external

```bash
kubectl apply -f app_lb.yml
```

Wait for the Loadbalancer to come up and Add the External IP
```bash
kubectl get svc --watch
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
app          LoadBalancer   10.100.22.121   <pending>     80:31084/TCP   25m
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        26m
mysql        ClusterIP      10.98.184.9     <none>        3306/TCP       23m

```


Test connectivity to app api pong 

```bash
kubectl exec -it dnsutils -- wget -qO- app:8080/ping 
```

Test connectivity to app api secret 

```bash
docker run -it gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 wget -qO- <EXTERNAL_IP>:80/secret 
```
```bash
docker run -it gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 wget -qO- <EXTERNAL_IP>:80/ping 
```

Both of these will fail since we have not allowed external traffic from the cluster

```bash
kubectl apply -f export_public.yml
```

Try again 

```bash
docker run -it gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 wget -qO- <EXTERNAL_IP>:80/secret 
```

```bash
docker run -it gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 wget -qO- <EXTERNAL_IP>:80/ping 
```

Finished! 

We will use this Cluster for the Troubleshooting DNS Lab in Lecture 7






 