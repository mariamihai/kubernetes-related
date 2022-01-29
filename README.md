# Kubernetes Tutorial

Following along with [Kubernetes Tutorial](https://www.youtube.com/watch?v=X48VuDVv0do&t=2s&ab_channel=TechWorldwithNana) from TechWorld with Nana.

<details>
    <summary>Table of Content</summary>

  - [Running locally](#running-locally)
    - [Installation and checking the version](#installation-and-checking-the-version)
  - [`minikube` commands](#minikube-commands)
  - [`kubectl` commands](#kubectl-commands)
    - [CRUD commands](#crud-commands)
    - [Create / delete via configuration file](#create--delete-via-configuration-file)
    - [Status](#status)
    - [Debugging pods](#debugging-pods)
    - [Working with namespaces](#working-with-namespaces)
  - [Demo project](#demo-project)
    - [Using secrets](#using-secrets)
    - [Create deployment and service for MongoDB](#create-deployment-and-service-for-mongodb)
    - [Using configMap](#using-configmap)
    - [Create deployment and service for Mongo Express](#create-deployment-and-service-for-mongo-express)

</details>

---

## Running locally

### Installation and checking the version

(for Windows)

<details>
    <summary>Install `kubectl` and `minikube`</summary>

```bash
choco install kubernetes-cli
kubectl version
kubectl version --client

choco install minikube
minikube version
```

</details>

<details>
    <summary>Start a cluster with a specific driver</summary>

```bash
minikube start --driver=hyperv
minikube start --vm-driver hyperv

minikube start --driver=docker

minikube start --vm-driver=hyperkit
```

</details>

([Hyperv](https://minikube.sigs.k8s.io/docs/drivers/hyperv/), [Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/))

<details>
    <summary>Set Docker as the default driver</summary>

```bash
minikube config set driver docker
```

</details>

---

## `minikube` commands

<details>
    <summary>commands</summary>

```bash
minikube start
minikube status
minikube stop

minikube delete
minikube delete all

# Assign IP address for external service
minikube service [service-name]
```

</details>

---

## `kubectl` commands

### CRUD commands

<details>
    <summary>commands</summary>

```bash
kubectl create deployment [deployment-name]
kubectl create deployment [deployment-name] --image=[image-name] [--dry-run] [options]

kubectl edit deployment [deployment-name]

kubectl delete deployment [deployment-name]
```

</details>

### Create / delete via configuration file

<details>
    <summary>commands</summary>

```bash
kubectl apply -f [file-name.yaml]

kubectl delete -f [file-name.yaml]
```

</details>
<br/>

### Status

<details>
    <summary>commands</summary>

```bash
kubectl get all
kubectl get all | grep [name]
kubectl get all -n [namespace]

kubectl get nodes

kubectl get namespaces

kubectl get pod
# Get more information about the mod
kubectl get pod -o wide
# Watch for changes
kubectl get pod --watch
# Get in yaml format
kubectl get pod -o yaml

kubectl get service

kubectl get replicaset

kubectl get deployment
# Check status
kubectl get deployment [deployment-name] -o yaml
# Save status
kubectl get deployment [deployment-name] -o yaml > result.yaml

kubectl get secret
```

</details>
<br/>

### Debugging pods

<details>
    <summary>commands</summary>

```bash
kubectl logs [pod-name]
lubectl exec -it [pod-name] -- bin/bash

kubectl describe pod [pod-name]
kubectl describe service [service-name]
```

</details>

</br>

### Working with namespaces

<details>
    <summary>Get cluster info from the kube-public namespace</summary>

```bash
kubectl cluster-info
```

</details>

<details>
    <summary>Create namespace</summary>

```bash
kubectl create namespace [namespace]
```

Can create via configuration file as well (preffered):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-configmap
  namespace: mongo-namespace
type: Opaque
data:
  database_url: mongo-service
```

</details>

<details>
    <summary>Check resources that are or not bound to a namespace</summary>

```bash
kubectl api-resources --namespaced=true

kubectl api-resources --namespaced=false
```

</details>

<details>
    <summary>Create resources in a specific namespace</summary>

```bash
kubectl apply -f file-name.yaml --namespace=[namespace]
```

Or via configuration file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-configmap
  # Define resource in namespace
  namespace: mongo-namespace
type: Opaque
data:
  database_url: mongo-service
```

</details>

<details>
    <summary>Get resources from a namespace</summary>

```bash
# If not specified then it returns the resources from the default namspace
kubectl get deployment [--namespace=default]

kubectl get deployment --namespace=[namespace]
kubectl get configmap --namespace=[namespace]
```

</details>

<details>
    <summary>Change active namespace with kubens</summary>

```bash
# Windows
choco install kubens

# Show existing namespaces and highlight the active one
kubens

# Sets this as the active namespace
kubens [different-namespace]

# Switch back to previous namespace
kubens -
```

Can have an [interactive mode](https://github.com/ahmetb/kubectx/#interactive-mode).

</details>

<details>
    <summary>Change contexts with kubectx</summary>

```bash
# Windows
choco install kubectx

# Show existing clusters and highlight the active one
kubectx

# Sets this as the active cluster
kubectx [different-cluster]

# Switch back to previous cluster
kubectx -

# Create an alias for the context
$ kubectx context=context_alias
```

Can have an [interactive mode](https://github.com/ahmetb/kubectx/#interactive-mode).

</details>

---

## Demo project

### Using secrets

<details>
    <summary>Encode username and password</summary>

```bash
echo -n 'secret' | base64
```

</details>

<details>
    <summary>Add the values to the mongo-secret.yaml file</summary>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-root-username: <base64 encoded>
  mongo-root-password: <base64 encoded>
```

</details>
<br/>

### Create deployment and service for MongoDB

<details>
    <summary>Reference the secret saved in the mongo-secret.yaml file in the deployment configuration file</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongo
spec:
  # ...
  template:
    # ...
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
        # Default port
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              # mongo-secret.yaml > metadata > name
              name: mongo-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              # mongo-secret.yaml > metadata > name
              name: mongo-secret
              key: mongo-root-password
```

</details>

<details>
  <summary>Create internal service for this deployment</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  # type not specified -> Internal service, ClusterIP type
  ports:
    - protocol: TCP
      # Service port
      port: 27017
      # Container / Pod port of deployment
      targetPort: 27017
```
</details>
<br/>

### Using configMap

<details>
    <summary>Add environment variables in .yaml file</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-configmap
data:
  database_url: mongo-service
```

</details>
<br/>

### Create deployment and service for [Mongo Express](https://hub.docker.com/_/mongo-express)

<details>
    <summary>Reference the secret saved in the mongo-secret.yaml file and the variable from configMap .yaml file in the deployment configuration file of Mongo Express</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express-deployment
  labels:
    app: mongo-express
spec:
  # ...
  template:
    # ...
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        # Default port
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME 
          valueFrom:
            secretKeyRef:
              # mongo-secret.yaml > metadata > name
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD  
          valueFrom:
            secretKeyRef:
              # mongo-secret.yaml > metadata > name
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER        
          valueFrom:
            configMapKeyRef:
              # mongo-configmap.yaml > metadata > name
              name: mongo-configmap
              key: database_url
```

</details>

<details>
  <summary>Create external service for this deployment</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  # Making it an external service
  type: LoadBalancer
  ports:
    - protocol: TCP
      # Service port
      port: 8081
      # Container / Pod port of deployment
      targetPort: 8081
      # Port for external IP address, must be between 30000 - 32767
      nodePort: 30000
```

</details>

<details>
  <summary>Assign external IP address when working with minikube</summary>

```bash
minikube service mongo-express-service
```

</details>

---

## Ingress

Example of using Ingress rules with dashboard.

There is a `kubernetes-dashboard` namespace specific to minikube use.

<details>
  <summary>Enable addons</summary>

```bash
minikube addons enable dashboard
minikube addons enable metrics-server
minikube addons list
```

Check the new namespace by running `kubectl get ns`.

</details>

<details>
  <summary>Get the dashboard in browser</summary>

```
minikube dashboard
minikube dashboard –url
```

</details>

<details>
  <summary>Install ingress controller in minikube</summary>

Using `K8s Nginx implementation of Ingress Controller`:

```bash 
minikube addons enable ingress
```

Check for `nginx-ingress-controller` pod:

```bash 
kubectl get pod -n ingress-nginx
```

Will need `minikube tunnel` to connect to LoadBalancer services.

</details>

<details>
  <summary>Configure an ingress rule for the dashboard</summary>

`ingress.yaml` file:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: dashboard.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: kubernetes-dashboard
              port:
                number: 80
```

```bash 
kubectl apply -f ingress.yaml
kubectl get ingress -n kubernetes-dashboard
```

The address and host needs to be set in `C:\Windows\System32\drivers\etc\hosts` (for Windows).

</details>


<details>
  <summary>TLS</summary>

Need to create a secret for the certificate:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret-tls
  namespace: default
data:
  tls.crt: based64 encoded cert
  tls.key: based64 encoded key
type: kubernetes.io/tls
```

And tie it to the Ingress yaml file:
```yaml
#...
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-secret-tls
#...
```

The data keys must be `tls.crt` and `tls.key`.

The values are the file contents and NOT the file path or locations.

The secret component must be in the same namespace as the Ingress component as you can’t reference a secret in another namespace.

</details>

<details>
  <summary>Links</summary>

https://kubernetes.github.io/ingress-nginx/deploy/#checking-ingress-controller-version

https://docs.nginx.com/nginx-ingress-controller/intro/how-nginx-ingress-controller-works/

https://stackoverflow.com/questions/70287043/run-ingress-in-minikube-and-its-address-shows-localhost

https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/

</details>

---

## Helm

Information added based on the official [quickstart guide](https://helm.sh/docs/intro/quickstart/).

<details>
  <summary>Install helm (v 3+ currently)</summary>

```bash
choco install kubernetes-helm
```

```bash
helm get -h
```

</details>


<details>
  <summary>Helm chart repository</summary>

Available Helm chart repositories in [Artifact Hub](https://artifacthub.io/packages/search?kind=0).

```bash
# Initializing a Helm chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# List the charts that can be installed
helm search repo bitnami
```

</details>

<details>
  <summary>Using charts</summary>

```bash
# Get the latest list of charts
helm repo update

# Install a chart
helm install bitnami/mysql --generate-name
help install my-release bitnami/mysql

# Wait for the pod
kubectl get pods --namespace default -w

# Get more info
helm show chart bitnami/mysql
helm show all bitnami/mysql
```

</details>

<details>
  <summary>Releases</summary>

```bash
helm list
heml ls

# Need --keep-history flag for release history to be kept after uninstall
helm status my-release

heml uninstall mysql-xxxxxx
heml uninstall my-release

helm rollback my-release
```

</details>

---
