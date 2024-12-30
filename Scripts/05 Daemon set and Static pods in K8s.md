
# Scenario 1

The company is using some software on their K8s worker nodes that periodically leaves unnecessary data in a particular location on the worker node. Since this software could run on any node at any time, this trash data periodically appears on various worker nodes and should be cleaned up.

Configure the cluster to create a pod on each worker node that periodically deleted the contents of the `/etc/beebox/tmp` on the worker node. Make sure a copy of this pod is automatically created on each worker node, even as new nodes are added to the cluster. Note that you should not run a copy of the pod on the control plane node.

## Creating a DaemonSet Specification YAML File

To create a DaemonSet descriptor:

```
vi daemonset.yml
```

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: beebox-cleanup
spec:
  selector:
	matchLabels:
	  app: beebox-cleanup
  template:
	metadata:
	  labels:
		app: beebox-cleanup
	spec:
	  containers:
	  - name: busybox
		image: busybox:1.27
		command: ['sh', '-c', 'while true; do rm -rf /beebox-temp/*; sleep 60; done']
		volumeMounts:
		- name: beebox-tmp
		  mountPath: /beebox-temp
	  volumes:
	  - name: beebox-tmp
		hostPath:
		  path: /etc/beebox/tmp
```

Then use apply to create the DaemonSet in the cluster:

```
kubectl apply -f daemonset.yml
```

To verify a DaemonSet pod is running on each worker node:

```
kubectl get pods -o wide
```

![Pasted image 20241213174219](https://github.com/user-attachments/assets/81bf7ed6-c150-43b6-a0df-126ff51d9c34)


# Scenario 2

The company has built a special diagnostic tool for its K8s nodes. This tool can be run as a container in a K8s Pod, and it collects detailed diagnostic data from the worker node throughout the node lifecycle.

One particularly useful feature is that this tool is able to collect data during the node startup process, before the kubelet begins communicating with the Kubernetes API, or even before a kubelet joins a cluster. To benefit from this, the Pod needs to run without depending on the presence of a Kubernetes API server connection. It will need to be run and managed directly by the kubelet.

Your task is to create a pod to run this diagnostic tool on the `Worker Node 1` server. Use the `acgorg/beebox-diagnostic:1` image for this Pod.

## Creating a Static Pod for diagnostic tool

In the `Worker Node 1` server we have create a static pod manifest file at `/etc/kubernetes/manifests`

```
sudo vi /etc/kubernetes/manifests/beebox-diagnostic.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: beebox-diagnostic
spec:
  containers:
  - name: beebox-diagnostic
	image: acgorg/beebox-diagnostic:1
	ports:
	- containerPort: 80
```


Then eventually kubelet will make it but we can restart kubelet to start the static pod instantly

```
sudo systemctl restart kubelet
```

To check the status of your static Pod, we log in to the `Control Plane Node` server:

```
kubectl get pods
```

Attempt to delete the static Pod using the k8s API:

```
kubectl delete pod beebox-diagnostic-k8s-worker1
```

Check the status of the Pod:

```
kubectl get pods
```

We'll see the Pod was immediately re-created, since it is only a mirror Pod created by the worker kubelet to represent the static Pod.

![Pasted image 20241213175727](https://github.com/user-attachments/assets/4a3b8e1b-696d-42a9-95f6-86b6c8e633b4)
