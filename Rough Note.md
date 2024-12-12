![[Pasted image 20241128212829.png]]

how to apply host name on a newly created server/vm 
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
```sudo apt-mark hold <package_name>```
This command is used to prevent package from auto updating, it's used to get some control on packages in your cluster

This command is used to get the join command in control plane for other nodes to join in our cluster with token
```kubeadm token create --print-join-command```

Ask about networking plugin calico

I used this command to initialize our cluster
```
 sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.27.11
```
here
`--pod-network-cidr` this is a ip range that is going to be used for our internal virtual pod network and for our networking plugin too

![[Pasted image 20241128225720.png]]

![[Pasted image 20241128230012.png]]

```
kubectl get pods
kubectl get namespaces
kubectl get pods --namespaces kube-system
kubectl get pods --all-namespaces
kubectl create nmaespaces <namespace_name>
```

# High Availability 
![[Pasted image 20241203132849.png]]
![[Pasted image 20241129121539.png]]
![[Pasted image 20241129121843.png]]
![[Pasted image 20241129121853.png]]
![[Pasted image 20241129122149.png]]


![[Pasted image 20241203163115.png]]
![[Pasted image 20241203163045.png]]![[Pasted image 20241203163058.png]]

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
    ```
    
    ```
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
![[Pasted image 20241129225431.png]]


# k8s Object management 
![[Pasted image 20241202213942.png]]

![[Pasted image 20241203190926.png]]
![[Pasted image 20241202220516.png]]
![[Pasted image 20241202220537.png]]
![[Pasted image 20241202220546.png]]
![[Pasted image 20241202220553.png]]
![[Pasted image 20241202220601.png]]


## kubectl tips
![[Pasted image 20241202220729.png]]
![[Pasted image 20241202220754.png]]
![[Pasted image 20241202220813.png]]

## Role Based Access Control (RBAC)
![[Pasted image 20241202222003.png]]![[Pasted image 20241202222015.png]]![[Pasted image 20241202222026.png]]

## Service Acc
![[Pasted image 20241202221700.png]]![[Pasted image 20241202221712.png]]![[Pasted image 20241202221729.png]]
## Inspecting pod resource usage
![[Pasted image 20241202222136.png]]
![[Pasted image 20241202222208.png]]



curl -sfL https://get.k3s.io | K3S_TOKEN=12345 sh -s - server --flannel-backend none --node-taint CriticalAddonsOnly=true:NoExecute --tls-san 35.171.22.24 


curl -sfL https://get.k3s.io | K3S_URL=https://35.171.22.24:6443 K3S_TOKEN=K1058e036383736221c0aa704a49045ee20f6c804065f23cad302f1df27f56f44ab::server:0fe6fb37148f5439db7421523db09a3f

curl -sfL https://get.k3s.io | K3S_URL=http://35.171.22.24:6443 K3S_TOKEN=K1058e036383736221c0aa704a49045ee20f6c804065f23cad302f1df27f56f44ab::server:0fe6fb37148f5439db7421523db09a3f sh -


curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --node-taint CriticalAddonsOnly=true:NoExecute --cluster-init --tls-san 35.171.22.24


curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --server https://35.171.22.24:6443 --tls-san 35.171.22.24




//Worked
curl -sfL https://get.k3s.io | K3S_URL=https://k3s_endpoint:6443 K3S_TOKEN=569b6470cd0a839fbff439aa65c696c7 --node-name k3s-worker-03 sh -
nano /etc/hosts

![[Pasted image 20241205182238.png]]


curl -sfL https://get.k3s.io | sh -s - server \
  --token=569b6470cd0a839fbff439aa65c696c7 \
  --tls-san=k3s_endpoint \
  --node-name k3s-master-02


//Worked
  curl -sfL https://get.k3s.io | K3S_TOKEN=569b6470cd0a839fbff439aa65c696c7 sh -s - server \
    --server https://172.31.85.51:6443 \
  --node-name k3s-master-03


------------



![[Pasted image 20241210195113.png]]


![[Pasted image 20241210192720.png]]
if they have to be tightly coupled then multi container is very good for those scenarios

how do different container interact with each other in a multi container setup 
![[Pasted image 20241210192842.png]]
when can you use this and example.


## Init containers like User data in EC2

![[Pasted image 20241210193930.png]]
![[Pasted image 20241210193943.png]]
![[Pasted image 20241210195229.png]]
![[Pasted image 20241210194339.png]]
![[Pasted image 20241210194448.png]]



## Pod Allocation

![[Pasted image 20241210200718.png]]
 ![[Pasted image 20241210200808.png]]
Scheduler can us various techniques to allot a pod in a node one of them is nodeSelector 
![[Pasted image 20241210200850.png]]

Or we can use nodeName to completely bypass scheduling and assign a pod to a node  


### Daemon Sets
![[Pasted image 20241210201512.png]]
![[Pasted image 20241210201534.png]]


### Static pods
![[Pasted image 20241210201902.png]]

mirror pod is a ghost representation of the static pod you can see it but you cant make changes to it
![[Pasted image 20241210201931.png]]


## Deployment 
![[Pasted image 20241210204140.png]]
![[Pasted image 20241210204217.png]]
![[Pasted image 20241210205144.png]]

to delete a pod made by a deployment you have to either change its desired state or delete the deployment cause deleting the pod will actually delete it but deployment will spin up a ew pod in that deleted pod's place

### Scaling Application with Deployment

![[Pasted image 20241210205756.png]]
![[Pasted image 20241210205902.png]]
![[Pasted image 20241210205853.png]]
![[Pasted image 20241210205946.png]]

## Rolling Update

![[Pasted image 20241210210413.png]]
![[Pasted image 20241210210428.png]]

# K8s Network

![[Pasted image 20241210213259.png]]
![[Pasted image 20241210213306.png]]

## CNI plugins
![[Pasted image 20241210213453.png]]
there are many cni plugin available 
![[Pasted image 20241210213526.png]]
![[Pasted image 20241210213545.png]]

## DNS in K8s
![[Pasted image 20241210213629.png]]
![[Pasted image 20241210213642.png]]
this is not very useful in when we are using pods cause dns name uses ip address and if we have that why can't we just use that, so these name will be useful when we will use services

## Network policy

![[Pasted image 20241210214156.png]]
![[Pasted image 20241210214206.png]]
![[Pasted image 20241210214232.png]]
![[Pasted image 20241210214302.png]]
![[Pasted image 20241210214328.png]]
![[Pasted image 20241210214351.png]]
