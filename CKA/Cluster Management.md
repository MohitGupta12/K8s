# Introduction to High Availability in K8s

High Availability in Kubernetes ensures that your applications and services remain accessible and operational even in the face of failures, that means to design cluster itself to be highly available and to do this we need multiple control plane 
![[Pasted image 20241203150140.png]]

So to give a brief overview this is what high availability setup would look like we would likely need a load balancer to communicate with k8s api and this also include communication with clients like kubelet on worker node
## How we handle etcd in a HA setup
So to handle etcd in a HA setup we mainly use these 2 patterns
### Stacked etcd
![[Pasted image 20241129121843.png]]
In Stacked etcd, etcd runs on the same node of control plane, so we are using kubeadm to make our clusters and kubeadm uses this pattern. so in a highly available setup each one of those control plane will have its own etcd.
### External etcd
![[Pasted image 20241129121853.png]]
In this we run etcd on a completely different node from that which are running our control plane, so in a high availability setup we would have any number of nodes that run our etcd that will communicate with our multiple control plane 


# K8s management Tools

These are some tools that interface with k8s to provide some additional functionality. 
### kubectl
Kubectl is the official command line interface for k8s and it will be your main way to work with k8s 
### kubeadm
Kubeadm is a tools to quickly and easily create your cluster 
### Minikube
Minikube is a tool to quickly set up your k8s for development purposes
It differ from kubeadm as in minikube is use to set up k8s in a single node or machine 
### Helm
Helm is a tool that provide templating and packaging for our k8s objects
We can use it to manage our own template or as helm calls it charts to easily manage complex configuration of multiple objects, we can also download and use shared charts
### Kompose
Kompose can translate Docker compose file into k8s objects.
We can use it to easily move our docker workflow to k8s
### Kustomize
It is a similar tools to helm as it allows you to manage configurations of different k8s tools 


# Safely Draining a K8s Node

When performing maintenance, we should be able to remove a node from cluster without causing any issue in our application.

Draining is a process where you gracefully remove all running containers from a specific node (potentially rescheduling them on another node) before performing maintenance tasks.

We can use this to safely drain a node
```
kubectl drain <node_name> --ignore-daemonsets
```
`--ignore-daemonsets` flag is used to remove pods that are tied to specifically to each node

When you use `kubectl drain` command it will implicitly cordons the node as part of the draining process
- Cordoning a node means to prevents new pods from being scheduled on the node
- This is what it will show when node in cordon![[Pasted image 20241203165954.png]]
So to uncordon them you can use 
```
kubectl uncordon <node_name>
```
so that new pods can be schedule on them again

# Upgrading K8s with Kubeadm

Draining is one of the processes performed while upgrading a node to keep our cluster up to date
When upgrading kubelet you have to reload the daemon and restart kubelet to start using upgraded version
## Lab Work
### For Control Plane Node
- Upgrade kubeadm on control plane node
- Drain the control plane node
- Plan the upgrade (using kubeadm upgrade plan)
- Apply the upgrade (using kubeadm upgrade apply)
- Upgrade Kubelet and kubectl on control plan node
- Uncordon the control plane node

### For Worker Node
- Drain the node
- Upgrade Kubeadm
- Upgrade kubelet configuration (kubeadm upgrade node)
- upgrade kubelet and kubectl
# Backing Up and Restoring etcd data

## Lab Work
### Back Up the `etcd` Data

1. Look up the value for the key `cluster.name` in the `etcd` cluster:    
    ```
    ETCDCTL_API=3 etcdctl get cluster.name \
      --endpoints=https://10.0.1.101:2379 \
      --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
      --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
      --key=/home/cloud_user/etcd-certs/etcd-server.key
    ```
    The returned value should be `beebox`.
    
2. Back up `etcd` using `etcdctl` and the provided `etcd` certificates:
    ```
    ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
      --endpoints=https://10.0.1.101:2379 \
      --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
      --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
      --key=/home/cloud_user/etcd-certs/etcd-server.key
    ```
    
3. Reset `etcd` by removing all existing `etcd` data:
    ```
    sudo systemctl stop etcd
    sudo rm -rf /var/lib/etcd
    ```

### Restore the `etcd` Data from the Backup

1. Restore the `etcd` data from the backup (this command spins up a temporary `etcd` cluster, saving the data from the backup file to a new data directory in the same location where the previous data directory was):
    ```
    sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
      --initial-cluster etcd-restore=https://10.0.1.101:2380 \
      --initial-advertise-peer-urls https://10.0.1.101:2380 \
      --name etcd-restore \
      --data-dir /var/lib/etcd
    ```
    
2. Set ownership on the new data directory:
    ```
    sudo chown -R etcd:etcd /var/lib/etcd
    ```
    
3. Start `etcd`:
    ```
    sudo systemctl start etcd
    ```
    
4. Verify the restored data is present by looking up the value for the key `cluster.name` again:
    ```
    ETCDCTL_API=3 etcdctl get cluster.name \
      --endpoints=https://10.0.1.101:2379 \
      --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
      --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
      --key=/home/cloud_user/etcd-certs/etcd-server.key
    ```
    The returned value should be `beebox`.