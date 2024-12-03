# Working with Kubectl

Kubectl is a command line tool that allows us to interact with k8s.
It can be used to deploy applications, inspect and manage cluster resources, and view logs
Some common commands would be
## Get 
```
kubectl get <objects_type> <object_name> -o <output> --sort-by <JSON_Path> --selector <selector> 
```
kubectl get : used to get objects in a cluster
- `-o` is used to set output format
- `--sort-by` is used to sort o/p using a JSON expression
- `--selector` is used to filter results by label
## Describe
```
kubectl describe <object_type> <object_name>
```
kubectl describe : used to get detailed information about a object 
## Create
```
kubectl create -f <file_name>
```
This is used to create a new object
`-f` can be used to supply a YAML file as a YAML descriptor

- If you attempt to create an object that has already exists, an error will occur
## Apply
```
kubectl apply -f <file_name>
```
This is similar to create however if you use apply on a object that already exists it will modify the existing object
## Delete
```
kubectl delete <object_type> <object_name>
```
This is used to delete a object
## Exec
```
kubectl exec <pod_name> -c <container_name> -- <command>
```
This is used to execute a command inside a container
This is very powerful while troubleshooting
But keep in mind container should have necessary software to run those command 

# Kubectl Tips 
## Imperative and declarative methods

![[Pasted image 20241203192511.png]]
Declarative methods is where you make object using YAML or JSON
- This is more commonly used then imperative
- This is used when we want to create object with some complex configurations

Imperative Method is when you use kubectl commands and flags
```
kubectl create deployment my-dep --image=nginx
```
- This is used when we want to create a quick and relatively simple object
## How to get a quick YAML sample

```
kubectl create deployment my-dep --image=nginx --dry-run -o yaml
```
we can use `--dry-run` flag to run a imperative command without creating an object and use `-o yaml` to quickly obtain a sample YAML file that you can edit 

## Recording a command

```
kubectl scale deployment my-dep replicas=5 --record
```
This `--record` flag will tell kubectl to record this command on its object
And when you use `kubectl describe` on that object it will show the recorded commands
# Managing Role Based Access Control

![[Pasted image 20241203194559.png]]

**Role Based Access Control** in k8 allows you to control what users are allowed to do and access within your cluster
for example you can give developer access to read data and logs from pods 

There are 2 types of roles
- Role : this is the default role that can allow user to do certain things within a namespace.
- ClusterRole : this is the role that can allow user to do certain things within a cluster

And both of the roles uses binding to connect with users roles use rolebinding and clusterRole uses clusterRoleBinding 

# Service Account

A Service Account is an account used by container processes within pod to authenticate with k8s API
It's essentially a way for pods to authenticate with the Kubernetes API server and perform actions on behalf of the pod
Commands to create Service account
```
kubctl create -f my-serviceaccount.yml
kubctl create sa my-serviceaccount.yml 
```

We can create service account with YAML
![[Pasted image 20241203195730.png]]
 
We can also manage access control for service account using RBAC,
it can be bind with any of the 2 bindings to connect with pods
![[Pasted image 20241203200005.png]]
we just have to put our service account in subjects

#  Inspecting Pod Resource Usage

In order to view metric about resources pod and container are using, we need to get a add-on to collect and provide that data, one of those is **K8s Metrics Server**

Once we have that add-on we can use
```
kubectl top pod --sort-by <JSON_Path> --seclector <selector>
```
with this we can view data about resource usage in our pods and nodes and we can also use `--sort-by` and `--selector` flags
![[Pasted image 20241203200742.png]]