
# Scenario 1
In first Scenario we have a pod that keep crashing, dev are looking into it but for time being we have to do something about it cause luckily crash can be fixed by simply rebooting the pod.
So in this case we have to make that pod into a self-healing one so that it restarts when it crashes.

## Set it to Restart the Container When It Is Down

Now we have to get yaml descriptor of the pod and Set the `restartPolicy` to `Always`

To get the pod's YAML descriptor we can use
```
  kubectl get pod beebox-shipping-data -o yaml > beebox-shipping-data.yml
```

To set the `restartPolicy` to `Always`:

```
vi beebox-shipping-data.yml
```

```
spec:
  ...
  restartPolicy: Always
  ...
```

## Add a Liveness Probe to Detect When the Application Has Crashed

Liveness probe:
```
spec:
  containers:
  - ...
	name: shipping-data
	livenessProbe:
	  httpGet:
		path: /
		port: 8080
	  initialDelaySeconds: 5
	  periodSeconds: 5
	...
```

Now we have to delete the existing pod and create a new pod with updated descriptor 

```
kubectl delete pod beebox-shipping-data
kubectl apply -f beebox-shipping-data.yml
```

We can check the http response from the pod (it will have a new IP address since we re-created it):

```
kubectl exec busybox -- curl <beebox-shipping-data_IP>:8080
```

And if application crashes and the pod is restarted automatically.

# Scenario 2
 In second scenario, we have our main container and it uses a shipping service. now our job is to make a init container that checks if shipping service is running or not and it should complete when service is running 

This is our main pod, 
```Initial pod config
apiVersion: v1
kind: Pod
metadata:
  name: shipping-web
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
```

we have to make a init container that does the checking part
```
apiVersion: v1
kind: Pod
metadata:
  name: shipping-web
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
  initContainers:
  - name: shipping-svc-check
    image: busybox:1.27
    command: ['sh', '-c', 'until nslookup shipping-svc; do echo waiting for shipping-svc; sleep 2; done']
```

Now our main app will be in "**init**" state until shipping-svc isn't running

Config for shipping service 
```
apiVersion: v1
kind: Service
metadata:
  name: shipping-svc
spec:
  selector:
    app: shipping-svc
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: shipping-backend
labels:
app: shipping-svc
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
```

Results after service is running:

![[Pasted image 20241213001511.png]]