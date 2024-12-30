
# Scenario 1

Currently, our company have two application components in two different namespaces: a user database and a web frontend.

Our developers aren't quite sure how Kubernetes DNS works, and they are having trouble reaching some of their Services as a result. They have asked you to perform some tests that will help confirm and clarify for them how Kubernetes DNS behaves.

A Pod called `busybox` already exists in the `web` namespace. You can use this Pod to perform your tests. Save the output of your tests to some files, so that the developers can see the results.

### Perform an Nslookup for a Service in the Same Namespace

We will start by using the `busybox` Pod in the `web` namespace to perform an nslookup on the `web-frontend` Service and save the results in a text file  by entering:

```
kubectl exec -n web busybox -- nslookup web-frontend

kubectl exec -n web busybox -- nslookup web-frontend >> ~/dns_same_namespace_results.txt
```

Now we will look up the same Service using the fully qualified domain name and save the results of the second nslookup in a text file by entering:

```
kubectl exec -n web busybox -- nslookup web-frontend.web.svc.cluster.local

kubectl exec -n web busybox -- nslookup web-frontend.web.svc.cluster.local >> ~/dns_same_namespace_results.txt
```

 Check that everything looks okay in the text file by using:

```
cat ~/dns_same_namespace_results.txt
```

![Pasted image 20241216000319](https://github.com/user-attachments/assets/2af28240-7326-4433-9be4-cffe246ad734)

we can see that there are both server and name address present that means we can use any type of service name to use that service if we are in same namespace

### Perform an Nslookup for a Service in the Different Namespace

Again we will uUse the `busybox` Pod in the `web` namespace to perform an nslookup on the `user-db` Service in the `data` namespace, while only utilizing the short Service name, by entering:
```
kubectl exec -n web busybox -- nslookup user-db
```

This first request is supposed to result in an error message.

Then save the results of this nslookup in a text file by using:
```
kubectl exec -n web busybox -- nslookup user-db >> ~/dns_different_namespace_results.txt
```

Second we will perform the same lookup using the fully qualified domain name and save the results in a text file by entering:

```
kubectl exec -n web busybox -- nslookup user-db.data.svc.cluster.local

kubectl exec -n web busybox -- nslookup user-db.data.svc.cluster.local >> ~/dns_different_namespace_results.txt
```

Check the output in the text file by using:
```
cat ~/dns_different_namespace_results.txt
```

![Pasted image 20241216000516](https://github.com/user-attachments/assets/8c66c282-01ef-4924-b281-5678ad65ed60)

we can see that there are only server and name address present for second nslookup that means we can use fully qualified domain type of service name to use that service if we are in different  namespace

# Scenario 2

A web authentication service, will need to be accessible by external clients using the BeeBox mobile app. A Deployment has been created to represent this Service (the app is still in early stages, so it is just running Nginx containers for now).

Create a ClusterIP service that will expose the `web-auth` Deployment. Then, create an Ingress which maps requests with the path `/auth` to the Service.
(For now, you do not need to worry about installing any Ingress controllers.)
### Create a Service to Expose the web-auth Deployment

First we will check out the deployment to get label on our Pods

```
kubectl get deployment web-auth -o yaml
```

Note the `web-auth` label on our Pods. We'll use this label to select these Pods using our Service.

Create a `web-auth-svc.yml` file:
```
vi web-auth-svc.yml
```

```
apiVersion: v1
kind: Service
metadata:
  name: web-auth-svc
spec:
  type: ClusterIP
  selector:
    app: web-auth
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
```

To create the Service:
```
kubectl create -f web-auth-svc.yml
```

### Create an Ingress That Maps to the New Service

To create a `web-auth-ingress.yml` file:
```
vi web-auth-ingress.yml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-auth-ingress
spec:
  rules:
  - http:
      paths:
      - path: /auth
        pathType: Prefix
        backend:
          service:
            name: web-auth-svc
            port:
              number: 80
```

To create the Ingress:
```
kubectl create -f web-auth-ingress.yml
```

Now to check the status of the Ingress:
```
kubectl describe ingress web-auth-ingress
```

![Pasted image 20241216002142](https://github.com/user-attachments/assets/7f6951f8-c650-4b89-a277-f54f004618e3)

Note the service endpoints in the Backends section of the output. That means we have successfully created an Ingress that maps to the backend Service.
