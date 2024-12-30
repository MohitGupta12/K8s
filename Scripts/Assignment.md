### The Challenge

Write a [FastAPI](https://fastapi.tiangolo.com/) server to create a highly available [K3s](https://k3s.io/) cluster on [Azure Virtual Machine](https://learn.microsoft.com/en-us/python/api/overview/azure/compute?view=azure-python) using just simple API endpoints. Deploy CloudNativePG in the K3s cluster using [Helm charts](https://helm.sh/).

#### Your task is to:

- Create a [K3s cluster with High Availability using Embedded etcd](https://docs.k3s.io/datastore/ha-embedded?_highlight=hig) for fault tolerance.
- Deploy the [CloudNativePG](https://github.com/cloudnative-pg/charts) to the K3s cluster using Helm charts. The PostgreSQL database should have an HTTP endpoint to connect.
- Configure DNS to point a custom domain name to the K3s cluster's load balancer IP.



**K3s Cluster Setup:**

1. **Install K3s:**
```
curl -sfL https://get.k3s.io | sh -s - server --node-name k3s-master-01
```


2. **Join Nodes to the Cluster:** 
- For Agent node
```
curl -sfL https://get.k3s.io | K3S_TOKEN K107f719c90198ea38021d2be4e03b9268eaf839c0530b84b5446b0a555399a2957::server:5452d9379bb78dcdd97431b9a067613e K3S_URL=https://172.31.23.9:6443 sh - 
```

- For Server node
```
curl -sfL https://get.k3s.io | K3S_TOKEN K107f719c90198ea38021d2be4e03b9268eaf839c0530b84b5446b0a555399a2957::server:5452d9379bb78dcdd97431b9a067613e K3S_URL=https://172.31.23.9:6443 sh -s - server
```


**Install Helm:**
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh
``````

**Add CloudNativePG Helm Repository:**
```
helm repo add cnpg https://cloudnative-pg.github.io/charts
```

**Deploy CloudNativePG:**
```
helm install cnpg cloudnativepg/cloudnativepg
helm upgrade --install database --namespace database --create-namespace cnpg/cluster
```

**Service to expose our database**
```cnpg-svc.yml
apiVersion: v1
kind: Service
metadata:
  name: cnpg-svc
spec:
  selector:
    cnpg.io/cluster: database-cluster
  ports:
  - protocol: TCP
    port: 5432
    targetPort: 5432
  type: LoadBalancer
```

```
kubectl apply -f cnpg-svc
```

**Install NGINX Ingress Controller:**
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm search repo ingress-nginx --versions
```

**Deploy Nginx Ingress Controller**
```
CHART_VERSION="4.4.0"
APP_VERSION="1.5.1"

mkdir ./kubernetes/ingress/controller/nginx/manifests/

helm template ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--version ${CHART_VERSION} \
--namespace ingress-nginx \
> ./ingress/controller/nginx/manifests/nginx-ingress.${APP_VERSION}.yaml

kubectl create namespace ingress-nginx
kubectl apply -f ./ingress/controller/nginx/manifests/nginx-ingress.${APP_VERSION}.yaml

```


**Create Ingress Resource:**
``` ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: db-ingress
  namespace: database
spec:
  ingressClassName: nginx
  rules:
  - host: db.com
    http:
      paths:
      - backend:
          serviceName: cnpg-svc
          servicePort: 5432 
```

```
kubectl apply -f ingress.yaml 
```
