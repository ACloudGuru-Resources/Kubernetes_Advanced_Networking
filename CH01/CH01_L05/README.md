# Tool Needed for A Cloud Guru Advance Network on Kubernetes in AWS Labs  

1.	VirtualBox
2.	Docker
3.	kubectl
4.	KIND (Kubernetes in Docker)
5.	aws cli
6.	eksctl

##	VirtualBox

VirtualBox is a general-purpose full virtualization for x86 hardware. 

#### PreReq

Check to make sure your laptop supports virtualization

Link to supported platforms

https://www.virtualbox.org/manual/UserManual.html#hostossupport

CLI example on Mac

`
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
`

#### Download

Download and install Virtualbox for your Operating System

https://www.virtualbox.org/wiki/Downloads

#### Install

Installation details 

https://www.virtualbox.org/manual/UserManual.html#installation

#### Verify

Download and run a Virtualbox image

There are many options when working with Virtualbox

- [Vagrant](https://app.vagrantup.com/boxes/search)
- [OS Boxes](https://www.osboxes.org/virtualbox-images/)
- Manual - Download a copy of an OS and [run through creating your first VM](https://www.virtualbox.org/manual/UserManual.html#intro-running)

###	[Docker](https://docs.docker.com/install/)

Docker is an open source container runtime

#### PreReq

- [Docker Hub account](https://hub.docker.com/sso/start)
- See the install for system requirements for your operating system.


#### Download
Download Docker for your operating system

- [Mac](https://docs.docker.com/docker-for-mac/)
- [Windows](https://docs.docker.com/docker-for-windows/)

#### Install

- [Mac](https://docs.docker.com/docker-for-mac/install/)
- [Windows](https://docs.docker.com/docker-for-windows/install/)

#### Verify

`docker --version`

`docker run hello-world`

##	kubectl

kubectl is the main entry point into the kubernetes API.  

#### PreReq

You must use a kubectl version that is within one minor version difference of your cluster. 
For example, a v1.2 client should work with v1.1, v1.2, and v1.3 master. Using the latest version of 
kubectl helps avoid unforeseen issues.

#### Download

#### Install

- [Mac](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)
- [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
- [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)

#### Verify

`kubectl version`

##	[KIND (Kubernetes in Docker)](https://kind.sigs.k8s.io/)

#### PreReq

- Docker

#### Download

https://kind.sigs.k8s.io/docs/user/quick-start

#### Install

https://kind.sigs.k8s.io/docs/user/quick-start

#### Verify

`kind create cluster`

##	aws cli

AWS CLI is one of the ways to programmatically access your AWS resources. 

#### PreReq

- Python
 
#### Download

See Install

#### Install

- [Mac/Linux](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux-mac.html)
- [Windows](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html)

#### Verify

`aws --version`

`aws-cli/1.16.290 Python/3.7.5 Darwin/19.2.0 botocore/1.13.26`

##	eksctl

eksctl is a CLI tool created by weave to help manage kubernetes clusters on AWS easily. 

#### PreReq

- AWS CLI 

#### Download

See Install

#### Install

https://eksctl.io/introduction/installation/

#### Verify

`eksctl version`

### Network Troubleshooting image 

Troubleshooting images can be difficult from outside the cluster, running a container with troubleshooting tools 
inside the cluster can be helpful. 

#### PreReq

- Docker

#### Download

Build it yourself, Dockerfile is located in this [repo here](https://github.com/strongjz/netshoot?organization=strongjz&organization=strongjz)

OR 

`docker pull acloudgurulabs/course_kubernetes_advanced_networking:latest`

<INSERT IMAGE HERE> 

#### Install

kubectl run --generator=run-pod/v1 tmp-shell --rm -i --tty --image acloudgurulabs/course_kubernetes_advanced_networking:latest -- /bin/bash

#### Verify

The Kubectl run should drop you into a shell

<INSERT IMAGE HERE> 

`ping google.com`
