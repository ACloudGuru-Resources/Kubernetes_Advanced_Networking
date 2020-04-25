# Troubleshooting Cluster DNS

### Objectives
1. Walk through some troubleshooting tips
2. Discuss failure modes 
3. Review resources/DNS outage stories

### Setup Steps
1. Deploy EKS Cluster
2. Deploy DNS utils images
3. DNS Resolution 
4. Walk Through failure modes
 
### 1. Deploy EKS Cluster 

If the cluster from Lecture 4 is not running, walk through the Deployment from [Ch03_L04 - Setting up an overlay network](../CH03_L04)
 
### 2. Deploy DNS utils images

Useful docker images that can be deployed to aid in troubleshooting dns issues 

```bash 
kubectl run --restart=Never -it --image  infoblox/dnstools dnstools 
```

dnsutils is used for [kubernetes end to end testing](https://github.com/kubernetes/kubernetes/tree/master/test/images)
 
```bash
kubectl run --restart=Never -it --image  gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 dnsutils
```
 or 
 
```bash
kubectl apply -f dnsutils.yml 
```
 
**Note**

Alpine based images may also have issues with search domains since it uses libmusl

This can be worked around this by using fully qualified names or migrate to debian-slim.

More details [here](https://github.com/gliderlabs/docker-alpine/blob/master/docs/caveats.md#dns), 
[here](https://github.com/coredns/coredns/issues/3391), and [here](https://gitlab.alpinelinux.org/alpine/aports/issues/9017)

# 3. DNS Resolution 

Deploying Hello Services from previous lecture

Hello Pod endpoints
```bash
kubectl apply -f deployment.yml
```

Hello ClusterIP Service
```bash
kubectl apply -f service.yml
```

Hello Headless Service
```bash
kubectl apply -f service-headless.yml
```

Resolve Hello service
```bash
kubectl exec -it dnsutils -- host -v -t a hello
```

Output

```bash
Trying "hello.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12359
# Please edit the object below. Lines beginning with a '#' will be ignored,
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;hello.default.svc.cluster.local. IN	A

;; ANSWER SECTION:
hello.default.svc.cluster.local. 30 IN	A	10.99.200.108

Received 96 bytes from 10.96.0.10#53 in 0 ms
```

Resolve Hello Headless service
```bash
kubectl exec -it dnsutils -- host -v -t a hello-headless

Trying "hello-headless.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28887
;; flags: qr aa rd; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;hello-headless.default.svc.cluster.local. IN A

;; ANSWER SECTION:
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.7
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.8
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.6
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.4
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.5
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.9
hello-headless.default.svc.cluster.local. 30 IN	A 10.244.0.10
```


Resolve google.com
```bash
kubectl exec -it dnsutils -- host -v -t a google.com
```

Output

```bash
Trying "google.com.default.svc.cluster.local"
Trying "google.com.svc.cluster.local"
Trying "google.com.cluster.local"
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47222
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		8	IN	A	216.58.193.142

Received 54 bytes from 10.96.0.10#53 in 0 ms
```

Add autopath @kubernetes and pods verified to the coredns configmap 
```bash
kubectl edit configmap coredns -n kube-system
```

Add the two highlight lines 
```yaml
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
*       pods verified
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
*    autopath @kubernetes
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

Redeploy the dnsutil image

```bash
kubectl delete po dnsutils
kubectl apply -f dnsutils
```

Resolve google.com again
```bash
kubectl exec -it dnsutils -- host -v -t a google.com
```

Output

```bash
Trying "google.com.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52185
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.default.svc.cluster.local. IN A

;; ANSWER SECTION:
google.com.default.svc.cluster.local. 30 IN CNAME google.com.
google.com.		30	IN	A	216.58.193.142

Received 140 bytes from 10.96.0.10#53 in 2 ms
```

# Failure modes 

### /etc/resolv.conf

1. Verify the /etc/resolv.conf 

```bash
kubectl exec -it dnsutils -- cat /etc/resolv.conf
```
Output 
```bash
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

The `nameserver` directive should match the ClusterIP of the CoreDNS Service, in this case 10.96.0.10

```bash
kubectl get svc kube-dns -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   15h
```

2. View all CoreDNS pods

```bash
kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o wide
```
Output
```bash
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE                  NOMINATED NODE   READINESS GATES
coredns-5644d7b6d9-42g9s   1/1     Running   0          16h   10.244.0.3   stack-control-plane   <none>           <none>
coredns-5644d7b6d9-mdgdb   1/1     Running   0          16h   10.244.0.2   stack-control-plane   <none>           <none>
```

3. Check to make sure coreDNS has endpoints 
```bash
kubectl get ep kube-dns --namespace=kube-system
NAME       ENDPOINTS                                               AGE
kube-dns   10.244.0.2:53,10.244.0.3:53,10.244.0.2:53 + 3 more...   16h
```

```bash
 kubectl describe ep kube-dns --namespace=kube-system
```

Output 2 Pods, with 3 ports exposed, 6 Endpoints
 
```bash
Name:         kube-dns
Namespace:    kube-system
Labels:       k8s-app=kube-dns
              kubernetes.io/cluster-service=true
              kubernetes.io/name=KubeDNS
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2020-03-22T02:55:44Z
Subsets:
  Addresses:          10.244.0.2,10.244.0.3
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    dns      53    UDP
    dns-tcp  53    TCP
    metrics  9153  TCP

Events:  <none>
```

5. Verify logs in CoreDNS

```bash
 for p in $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name); do kubectl logs --namespace=kube-system $p; done
```
Output
```bash
○ →  for p in $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name); do kubectl logs --namespace=kube-system $p; done
.:53
2020-03-22T02:55:38.525Z [INFO] plugin/reload: Running configuration MD5 = f64cb9b977c7dfca58c4fab108535a76
2020-03-22T02:55:38.525Z [INFO] CoreDNS-1.6.2
2020-03-22T02:55:38.525Z [INFO] linux/amd64, go1.12.8, 795a3eb
CoreDNS-1.6.2
linux/amd64, go1.12.8, 795a3eb
.:53
2020-03-22T02:55:38.533Z [INFO] plugin/reload: Running configuration MD5 = f64cb9b977c7dfca58c4fab108535a76
2020-03-22T02:55:38.533Z [INFO] CoreDNS-1.6.2
2020-03-22T02:55:38.533Z [INFO] linux/amd64, go1.12.8, 795a3eb
CoreDNS-1.6.2
linux/amd64, go1.12.8, 795a3eb
```

**Known Issue with CoreDNS and /etc/resolv.conf** 

Some Linux distros use systemd-resolved and it replaces /etc/resolv.conf to /run/systemd/resolve/resolv.conf

This can be fixed manually by using kubelet’s --resolv-conf flag to point to the correct resolv.conf 

https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/#known-issues


### CoreDNS Resources

[Total DNS outage due to OOM Killer](https://github.com/zalando-incubator/kubernetes-on-aws/blob/dev/docs/postmortems/jan-2019-dns-outage.md)

[Memory Limits and requests](https://github.com/coredns/deployment/blob/master/kubernetes/coredns.yaml.sed#L117) 

Default deployment memory requests and limits 

```yaml
        resources:
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
```

View the current deployments memory limits and requests

```bash
kubectl get pods -l k8s-app=kube-dns -n kube-system -o=jsonpath="{range .items[*]}[{.metadata.name}, {.spec.containers[*].resources.limits.memory},{.spec.containers[*].resources.requests.memory} ] {end}"
```


Autoscaler 

https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns-horizontal-autoscaler

https://kubernetes.io/docs/tasks/administer-cluster/dns-horizontal-autoscaling/

```
apiVersion: autoscaling/v1
   kind: HorizontalPodAutoscaler
   metadata:
     name: coredns
     namespace: default
   spec:
     maxReplicas: 20
     minReplicas: 2
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: coredns
     targetCPUUtilizationPercentage: 50
```
### Monitoring

Ensure you are monitoring and alerting on CoreDNS Metrics 

https://coredns.io/plugins/metrics/

CoreDNS exports process and Go runtime metrics as well as CoreDNS-specific metrics

Set Thresholds on items such 

`coredns_dns_request_duration_seconds` - duration to process each query.

`coredns_dns_request_count_total` - Counter of DNS requests made per zone, protocol and family

Also on pod level metrics 

`process_resident_memory_bytes` - Resident memory size in bytes

Many of the CoreDNS plugins also export metrics when the Prometheus plugin is enabled. 

Example: 

[Health plugin exports](https://coredns.io/plugins/health/) 

`coredns_health_request_duration_seconds{}` - duration to process a HTTP query to the local /health endpoint. 
As this a local operation it should be fast. A (large) increase in this duration indicates the CoreDNS process 
is having trouble keeping up with its query load.

### priorityClassName

Ensure that your CoreDNS deployment has the priority class set 

[It is set in the default deployment yaml](https://github.com/coredns/deployment/blob/master/kubernetes/coredns.yaml.sed#L96)

Verify Priority Class Name 

```bash
kubectl get pods -l k8s-app=kube-dns -n kube-system -o=jsonpath="{range .items[*]}[{.metadata.name},{.spec.priorityClassName} ] {end}"
```

### CoreDNS Plugins

These will also help see what is going on inside CoreDNS

[Error](https://coredns.io/plugins/errors/) - Any errors encountered during the query processing will be printed to 
standard output.

[Trace](https://coredns.io/plugins/trace/) - enable OpenTracing of how a request flows through CoreDNS

[Log](https://coredns.io/plugins/log/) - query logging

[health](https://coredns.io/plugins/health/) - CoreDNS is up and running this returns a 200 OK HTTP status code

[ready](https://coredns.io/plugins/ready/) - By enabling ready an HTTP endpoint on port 8181 will return 200 OK when 
all plugins that are able to signal readiness have done so.

**Only use for Testing and NOT in production** 

[debug](https://coredns.io/plugins/debug/) - debug disables the automatic recovery upon a crash so that you’ll get a 
nice stack trace.

Ready and Health should be used in the deployment

        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 8181
            scheme: HTTP

View current coreDNS corefile 

```bash
kubectl describe cm coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}

Events:  <none>
```

Make sure to delete this cluster

```bash
eksctl delete cluster --name test-cluster
[ℹ]  eksctl version 0.12.0
[ℹ]  using region us-west-2
[ℹ]  deleting EKS cluster "test-cluster"
[ℹ]  account is not authorized to use Fargate. Ignoring error
[✔]  kubeconfig has been updated
[ℹ]  cleaning up LoadBalancer services
[ℹ]  2 sequential tasks: { delete nodegroup "ng-d4923ffb", delete cluster control plane "test-cluster" [async] }
[ℹ]  will delete stack "eksctl-test-cluster-nodegroup-ng-d4923ffb"
[ℹ]  waiting for stack "eksctl-test-cluster-nodegroup-ng-d4923ffb" to get deleted
[ℹ]  will delete stack "eksctl-test-cluster-cluster"
[✔]  all cluster resources were deleted

```
            
# Resources

* [10 Ways to Shoot Yourself in the Foot with Kubernetes with Datadog](youtube.com/watch?v=QKI-JRs2RIE)
* [DNS outages caused by OOM Issues](https://github.com/zalando-incubator/kubernetes-on-aws/blob/dev/docs/postmortems/jan-2019-dns-outage.md)
* [Ndot issues](https://www.adammargherio.com/a-perfect-dns-storm/)
* [More ndot issues](https://pracucci.com/kubernetes-dns-resolution-ndots-options-and-why-it-may-affect-application-performances.html)
* [K8s.io debugging DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
* [K8s.io Guaranteeing Critical pods](https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/)
* [k8s.io Node OOM Behavior](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/#node-oom-behavior)
* [EKS DNS at scale and spikeiness](https://aws.amazon.com/blogs/containers/eks-dns-at-scale-and-spikeiness/)
* [CoreDNS Graphana Dashboards](https://grafana.com/grafana/dashboards/7279)
