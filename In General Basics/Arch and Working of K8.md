Kubernetes is a Open Source container Orchestration platform
- it automates deployment, scaling and management of containerized applications
- It can be traced back to Google's internal container orchestration system borg, and in 2014 they open sourced a version of borg named as Kubernetes
	![[Pasted image 20241126224437.png]]
- what is K8s
	8 stand for 8 letters between k and s and yeah this is bit nerdy 
	![[Pasted image 20241126224647.png]]

# Architecture 
A K8 cluster is a set of machine called nodes and these nodes are used to run containerized applications
![[Pasted image 20241127103526.png]]




# Storage

Volume can be mounted to a pod on a specific path and each pod has a unique path to same volume
- Empty Dir is a temporary volume that is on the machine where pod is located
- Persistent Volume and Persistent Volume Claims

# Scaling
K8s has mainly 3 types of auto Scaler to meet the increase or decrease in demand 
- Horizontal Pod Autoscaler - Increase/decrease number of pods in a node
- Vertica Pod Autoscaler - Increase/decrease resources of pods in a node
- Cluster Autoscaler - Increase/decrease number of nodes in cluster

![[Pasted image 20241127105204.png]]