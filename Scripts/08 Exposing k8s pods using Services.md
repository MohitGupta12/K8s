
Our Company have a backend database of user information and a web frontend, both of which are managed using deployments. Both of these applications are also still being developed and are currently just using simple Nginx containers for testing. You have been asked to set up appropriate Kubernetes Services for these application components.

The user database is a backend service that should only be accessible by other components within the cluster. The web frontend needs to be accessible by users outside the cluster. Locate the existing deployments and create the necessary Services to expose them. There is an existing Pod called `busybox` which you can use to test Services.
### Exposing the Pods from the User-db Deployment as an Internal Service

First we will examine the properties of the `user-db` deployment by using `kubectl get deployment user-db -o yaml`.

In the deployment properties, we will found labels `app: user-db` and ports

To create a Service descriptor we can use 
```
nano user-db-svc.yml
```

```
apiVersion: v1
kind: Service
metadata:
  name: user-db-svc
spec:
  type: ClusterIP
  selector:
    app: user-db
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

And create Service by using 
```
kubectl create -f user-db-svc.yml
```

Then we can test the Service to make sure it works by using 
```
kubectl exec busybox -- curl user-db-svc
```
![Pasted image 20241215233914](https://github.com/user-attachments/assets/51a9d1cb-c777-4056-8879-d9e5aafe69a4)


### Exposing the Pods from the Web-frontend Deployment as an External Service

We will again, examine the properties of the frontend deployment by using 
```
kubectl get deployment web-frontend -o yaml
```

 In Pod template, we will see the label `app=web-frontend`and exposed ports

To create a Service descriptor we can use 
```
nano web-frontend-svc.yml
```

```
apiVersion: v1 
kind: Service 
metadata: 
  name: web-frontend-svc 
spec: 
  type: NodePort 
  selector: 
	app: web-frontend 
  ports: 
  - protocol: TCP 
	port: 80 
	targetPort: 80 
	nodePort: 30080
```

To create the Service by using 
```
kubectl create -f web-frontend-svc.yml
```

To test the Service from outside the cluster we can use browser by navigating to `http://<PUBLIC_IP_ADDRESS>:30080`.

![Pasted image 20241215234428](https://github.com/user-attachments/assets/68913bdc-ba37-499c-b52d-3c6efcf53213)
