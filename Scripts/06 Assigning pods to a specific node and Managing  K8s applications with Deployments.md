# Scenario 1
 The company we work in has a few pods running in their Kubernetes cluster that depend on special services that exist outside the cluster. These services are highly sensitive, and the security team has asked that they be exposed only to certain network segments.

Unfortunately, only the `k8s-worker2` node exists in the network segment shared by these services. This means only pods on the `k8s-worker2` node will be able to access these sensitive external services, and pods on the `k8s-worker1` or `k8s-control` nodes cannot access them.

Your task is to reconfigure the `auth-gateway` pod and the `auth-data` deployment's replica pods so they will always run on the `k8s-worker2` node.
![[Pasted image 20241213121754.png]]

So in this case we have to schedule pods only on `k8s-worker2` so that they can access external services, for this we can either use labels and nodeSelector to allot these pod to that node or we could use NodeName directly either works

Here we will use labels,
![[Pasted image 20241213122638.png]]

First We have to add a Label to `k8s-worker2`, for this we can use :
```
kubectl label nodes k8s-worker2 external-auth-services=true
```
![[Pasted image 20241213123120.png]]

## For `auth-gateway pod`

For this we have to make a new descriptor that will have nodeSelector and then create a new pod with that updated descriptor
```
kind: Pod
metadata:
  name: auth-gateway
  namespace: beebox-auth
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    ports:
    - containerPort: 80
  nodeSelector:
    external-auth-services: "true"
```

now we have to delete old pod and create new pod with this descriptor
```
kubectl delete pods auth-gateway -n beebox-auth
kubectl apply -f auth-gateway.yml -n beebox-auth
```

![[Screenshot 2024-12-13 124142.png]]

## For `auth-data Deployment`

For the Deployment we have to update the descriptor and use `apply` on it and it will udate the deployment

To update the descriptor, add nodeSelector to it 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-data
  namespace: beebox-auth
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-data
  template:
    metadata:
      labels:
        app: auth-data
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
      nodeSelector:
        external-auth-services: "ture"
```

Update the deployment :
```
kubectl apply -f auth-data.yml -n beebox-auth
```

Results : 
![[Pasted image 20241213130204.png]]

# Scenario 2

Our company has many application under its belt. Unfortunately, there are some problems with the app. Two steps will need to be taken to fix this issue.
First, you will need to deploy a newer version of the app (`1.0.2`) that contains some performance improvements from the developers. 
Second, you will need to scale up the app deployment, increasing the number of replicas from `2` to `5`

To edit our deployment we can use 
```
kubectl edit deployment.v1.apps/beebox-web
```

Then locate the Pod's container specification, and change the `1.0.1` image version tag to `1.0.2`:
```
...

spec:
  containers:
  - image: acgorg/beebox-web:1.0.2
	imagePullPolicy: IfNotPresent
	name: web-server

...
```

We can check the status of our deployment to watch the rolling update 
```
kubectl rollout status deployment.v1.apps/beebox-web
```

And to scale up, we can do it by editing our descriptor or we can use `scale`:
```
kubectl scale deployment.v1.apps/beebox-web --replicas=5
```
