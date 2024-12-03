# Basics

**Define K8s** : Kubernetes is a portable, extensible and open source platform to manage containerized workload and services, that facilitates both declarative configurations and automation 
![[Pasted image 20241127110931.png]]

Primary Features
- Container Orchestration : Dynamically managing containers across multiply host systems
- Application Reliability : K8s makes is easier to make reliable, self-healing and scalable applications
- Automation : It has variety of features to help us automate our management

## Architecture 

A K8 cluster is a set of machine called nodes and these nodes are used to run containerized applications
![[Pasted image 20241128212829.png]]

There are 2 main parts of a K8 cluster 
- Control Plane
	- It makes global decisions about the cluster
	- API Server is responsible to communicate with worker node 
	- ETCD is a distributed key vale based storage system
	- Scheduler is used to schedule pods and make placement decision
	- Controller Manager is responsible for many thing one of them is managing state of cluster
- Worker nodes 
	- Kubelet is the daemon that runs on each worker node and it communicate with api server to maintain desired state of the pod 
	- Container runtime (containerD) is responsible for running and managing resources of container
	- Kube-proxy is a networking proxy and routes traffic to correct path and it also provide load balancing

### Lab Work
#### how to apply host name on a newly created server/vm 
On the node:
```sudo hostnamectl set-hostname <node_name>```

On all nodes, set up the hosts file to enable all the nodes to reach each other using these hostnames:
``sudo vi /etc/hosts``

On all nodes, add the following at the end of the file. You will need to supply the actual private IP address for each node:
```
<control plane node private IP> k8s-control
<worker node 1 private IP> k8s-worker1
<worker node 2 private IP> k8s-worker2
```

Log out of all three servers and log back in to see these changes take effect.
#### "tee" command
 The `tee` command in Linux is a powerful tool for splitting the output of a command and sending it to both the standard output (usually your terminal) and one or more files simultaneously.
#### apt-mark hold
```
sudo apt-mark hold <package_name>
```
This command is used to prevent package from auto updating, it's used to get some control on packages in your cluster
#### Cluster Initializing 
I used this command to initialize our cluster
```
 sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.27.11
```
here
`--pod-network-cidr` this is a ip range that is going to be used for our internal virtual pod network and for our networking plugin too
#### Cluster Join cmd
```
kubeadm token create --print-join-command
```
This command is used to get the join command in control plane for other nodes to join in our cluster with token

## Namespaces

Namespaces are virtual clusters backed by the same physical cluster.
Imagine a large apartment complex. Each apartment is a separate unit, housing different tenants. While they share the same building structure (the cluster), they have their own private space and resources.

In Kubernetes, **namespaces** are similar to these apartments. They provide a way to logically divide a single cluster into multiple virtual clusters. This allows different teams or projects to share the same physical cluster without interfering with each other

## Lab Work

- These are some commands to create a namespace and to get pods of a particular namespace or all namespaces
	```
	kubectl get pods
	kubectl get namespaces
	kubectl get pods --namespaces kube-system
	kubectl get pods --all-namespaces
	kubectl create nmaespaces <namespace_name>
	```
- If you don't specify any particular namespace, kubectl will assume you mean default namespace  