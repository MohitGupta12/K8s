# Creating Secret and Config Map

## Secret
First to create a secret will we need some data to put in it, for this we can use "htpasswd"
this will give us a encrypted password
```bash
htpasswd -c .htpasswd mohit
cat .htpasswd 
```
- password : user123 
```output
mohit:$apr1$7f86.a06$g7Q3J1463p1139q107u1
```

Then we can create a generic secret with this 
```
kubectl create secret generic nginx-htpasswd --from-file .htpasswd
```

## Config Map 

We can create a Config map that will take its data from a file or we can provide it in the CLI
```
nano app-config.yml
```

```app-config.yaml
database_host: my-database 
database_port: 5432 
```

And to create the Config Map we can use   
```
kubectl create configmap my-app-config --from-file=app-config.yml
```

And we could see our config-map with "describe"
```
kubectl describe configmap my-app-config
```

# Mounting it to the Pod

In order to mount this we have add these 2 in the volume in the pod configuration file 
```
nano pod.yml
```

```
apiVersion: v1 
kind: Pod metadata: 
name: nginx 
spec: 
	containers: 
		- name: nginx 
		  image: nginx:1.19.1 
		  ports: 
			- containerPort: 80 
			volumeMounts: 
			  - name: config-volume 
				mountPath: /etc/nginx 
			  - name: htpasswd-volume 
				mountPath: /etc/nginx/conf 
	volumes: 
		- name: config-volume 
		  configMap: 
			name: my-app-config 
		- name: htpasswd-volume 
		  secret: 
			secretName: nginx-htpasswd
```

Now to create this pod
```
kubectl apply -f pod.yml
```

Then we could use command to get our nginx welcome page 
```
kubectl exec busybox -- curl -u mohit:user123 <NGINX_POD_IP>
```