Kubernetes is a Open Source container Orchestration platform
- it automates deployment, scaling and management of containerized applications
- It can be traced back to Google's internal container orchestration system borg, and in 2014 they open sourced a version of borg named as Kubernetes
![Pasted image 20241126224437](https://github.com/user-attachments/assets/86b4ac5d-6f05-4707-8901-c7c32fb1252b)

- what is K8s
	8 stand for 8 letters between k and s and yeah this is bit nerdy 
	![Pasted image 20241126224647](https://github.com/user-attachments/assets/0bdec08f-7f83-4562-9150-212833811be8)


# Architecture 
A K8 cluster is a set of machine called nodes and these nodes are used to run containerized applications
![Pasted image 20241127103526](https://github.com/user-attachments/assets/f70dee7e-18f9-4158-9993-dae72e8efdd8)

There are 2 main parts of a K8 cluster 
- Control Plane
	- It makes global decisions about the cluster
	- API Server is responsible to communicate with worker node 
	- ETCD is a distributed key vale based storage system
	- Scheduler is used to schedule pods and make placement decision
	- Controller Manager is responsible to manage state of cluster
- Worker nodes 
	- Kubelet is the daemon that runs on each worker node and it communicate with api server to maintain desired state of the pod 
	- Container runtime is responsible for running and managing resources of container
	- Kube-proxy is a networking proxy and routes traffic to correct path and it also provide load balancing


# Storage

Volume can be mounted to a pod on a specific path and each pod has a unique path to same volume
- Empty Dir is a temporary volume that is on the machine where pod is located
- Persistent Volume and Persistent Volume Claims

# Scaling
K8s has mainly 3 types of auto Scaler to meet the increase or decrease in demand 
- Horizontal Pod Autoscaler - Increase/decrease number of pods in a node
- Vertica Pod Autoscaler - Increase/decrease resources of pods in a node
- Cluster Autoscaler - Increase/decrease number of nodes in cluster

![Pasted image 20241127105204](https://github.com/user-attachments/assets/80009378-895b-42e3-817c-70c7d257415f)
