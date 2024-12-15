Our company has set up a new development cluster to be used by an external contractor's development team. The cluster seems to be working and the team is able to access it, but they are reporting there is an issue.

You have received the below email describing the problem. Fix the issue, and verify pods can communicate with one another using the Kubernetes network.

```
Hello,
We've been trying to set up a network connection between two pods, called "cyberdyne-frontend" and "testclient". However, whenever we try to create these pods, they never get up and running. Can you look into this and figure out what is going on? Once the pods are up and running, we need you to verify they can communicate via network as well.

Thanks for your help!

Brando Smith
CyberDyne Systems
```

### Fix the Issue Causing Pods Not to Start Up

Trying to find issues
```
kubectl get pods
kubectl get nodes
```
![Pasted image 20241213213947](https://github.com/user-attachments/assets/178d89a5-7183-4339-8c8f-a4d9a10b1c30)

It looks like the nodes are `NotReady`.

Lets Describe a node to get more info
```
kubectl describe node k8s-worker1
```
It looks like `kube-proxy`, a component that handles networking-related tasks, is stuck starting up.

Now lets check the status of the networking plugin pods:
```
kubectl get pods -n kube-system
```
The networking pods seem to be missing. Most likely, a networking plugin was never installed.

To solve this lets install the Calico networking plugin:
```
kubectl apply -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
```

After this lets check the status of the Nodes and Pods again:

```
kubectl get nodes
kubectl get pods
```
They should both be `Ready` after about a minute.
![Pasted image 20241213214021](https://github.com/user-attachments/assets/32a99e8c-cef4-4e35-98cf-28da7da0aedd)


### To verify that you can Communicate between Pods Using the Cluster Network

First lets, verify the two pods can communicate over the network:
```
kubectl get pods -o wide
```

2. Run `curl` on the IP address of the `cyberdyne-frontend` Pod (which will be listed in the output from the previous command):

```
kubectl exec testclient -- curl <cyberdyne-frontend_POD_IP>
```

The result should be HTML of an Nginx page, meaning the Pods are able to communicate.
![Pasted image 20241213213907](https://github.com/user-attachments/assets/02abd4d7-4d71-44be-97b2-a2e218dcdb2d)

