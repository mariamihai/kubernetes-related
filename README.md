# Kubernetes related resources

## FreeCodeCamp tutorial - Kubernetes Course

Follow along for [the course](https://www.youtube.com/watch?v=d6WC5n9G_sM&t=2710s&ab_channel=freeCodeCamp.org) by Bogdan Stashchuk.

## TechWorld with Nana

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

kubectl [overview](https://kubernetes.io/docs/reference/kubectl/), 
[cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/),
[conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)

```bash
kubectl explain pod
```

### Grouped commands

<details>
    <summary>POD, ReplicaseSet, Deployment</summary>
    
#### POD related

Definition yaml file:
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels:
		app: myapp
		type: frontend
spec:
containers:
          - name: nginx-container
	image: nginx
```

```bash
kubectl create -f pod-definition.yaml
kubectl create -f pod-definition.yaml --dry-run=client [-o json / yaml / name]
kubectl run nginx --image=nginx [--restart=Never] 
kubectl run custom-nginx --image=nginx --port=8080
kubectl run redis --image=redis:alpine --dry-run=client --labels="tier=db" -o yaml > redis-pod.yaml
kubectl run webapp-green –image=kodekloud/webapp-color -- –color=green 

# Create pod and a service of type ClusterIp with the same name with target port for the service=80
kubectl run httpd --image=httpd:alpine --port=80 --expose

kubectl get pods

kubectl describe pod myapp-pod

kubectl delete pod myapp-pod
kubectl delete pod p1 p2 p3 ...

kubectl get pod myapp-pod -o yaml > pod-definition.yaml

# Get environment variables on a POD
kubectl exec webapp-color env
```

#### ReplicaSet related

Use `replicaset` or `rs`.

Definition yaml file:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: frontend
spec:
  template:
    # Pod definition here:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 2
  # Difference between RC and RS - selected is required
  selector:
    matchLabels:
      type: frontend
```

```bash
kubectl create -f replicaset-definition.yaml
kubectl create -f replicaset-definition.yaml --dry-run=client [-o json / yaml / name]
kubectl delete -f replicaset-definition.yaml

kubectl get replicationset

kubectl describe replicaset myapp-replicas
kubectl delete replicaset myapp-replicas
kubectl delete replicaset rs1 rs2 rs3 ...

kubectl get replicaset myapp-replicas -o yaml > replicaset-definition.yaml

# Change the number of replicas - pods are created or deleted automatically
kubectl replace -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset myapp-replicas

# Either delete and recreate the replicaset or delete the pods (if changing the image for example):
kubectl edit replicaset myapp-replicas

```

#### Deployment related

Definition yaml file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels:
		app: myapp
		type: frontend
spec:
	template:
		# Pod definition here:
	    metadata:
	        name: myapp-pod
	        labels:
		        app: myapp
		        type: frontend
        spec:
            containers:
                - name: nginx-container
	            image: nginx
	replicas: 3
	# Difference between RC and RS - selected is required
	selector:
		matchLabels:
			type: frontend
```

```bash
kubectl create -f deployment-definition.yml
kubectl create -f deployment-definition.yaml --dry-run=client [-o json / yaml / name]
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3

kubectl get deployments

kubectl get all
```

#### Namespace related

Use `namespaces` or `ns`.

Definition yaml file:
```yaml
apiVersion: v1
kind: Namespace
metadata:
	name: dev
```

```bash
kubectl get namespaces
kubectl get pods  --namespace=kybe-system


kubectl create -f namespace-definition.yml
Kubectl create namespace dev


kubectl create -f pod-definition.yml --namespace=dev
# or create the pod definition with namespace metadata
```

##### Switch to another namespace permanently

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
#kubectl get pods --namespace=default

kubectl get pods --all-namespaces
kubectl get pods -A

 kubectl get all --namespace=kube-system
 kubectl get all -n=kube-system
```

##### Limit resources in a namespace

Definition yaml file:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
	name: compute-quota
	namspeace: dev
spec:
	hard:
		pods: “10”
		requests.cpu: “4”
		requests.memory: 5Gi
		limits.cpu: “10”
		limits.memory: 10Gi
```

```bash
kubectl create -f compute-quota.yml
```

#### Service related

```bash
## Create a service redis-service to expose the redis application within the cluster on port 6379
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
# (This will automatically use the pod's labels as selectors)

kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
#(This will not use the pods labels as selector)

kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
# (This will automatically use the pod's labels as selectors)

kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
# (This will not use the pods labels as selectors)
```

#### ConfigMap related

pod-definition.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
specs:
    containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	     - containerPort: 8080
	# Env vars with CONFIGMAPS
	  env:
	     - name: APP_COLOR
	       valueFrom: 
		configMapKeyRef:
```

Create a ConfigMap - imperative way:
```bash
kubectl create configmap \
	app-config --from-literal=APP_COLOR=blue \
		   --from-literal=APP_MOD=prod


kubectl create configmap \
	app-config --from-file=app_config.properties
```

Create a ConfigMap - declarative way via a definition file:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: app-config
data:
	APP_COLOR: blue
	APP_MOD: prod
```

```bash
kubectl get configmaps
kubectl get cm
```

#### Secrets related

##### Create secrets

Imperative way: 
```
kubectl create secret generic \
	<secret-name> --from-literal=<key>=<value>

kubectl create secret generic \
	<secret-name> --from-file=<path-to-file>
```

Declarative way: 
```
kubectl create -f secret-data.yaml
```

Create a Secret - declarative way via a definition file:
```yaml
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data:
	key: <encoded-value>
```

Encode the data with `echo -n ‘value-to-be-encoded’ | base64`.

Decode it with `echo -n ‘encoded-value’ | base64 –-decode`.


```
kubectl get secrets

# Show the attributes in the secrets but hide the values:
kubectl describe secrets

# Get the values (encoded):
kubectl get secret app-secret -o yaml

```



</details>

### CRUD commands

<details>
    <summary>commands</summary>

```bash
kubectl create deployment [deployment-name]
kubectl create deployment [deployment-name] --image=[image-name] [--dry-run] [options]

kubectl edit deployment [deployment-name]

kubectl delete deployment [deployment-name]

# all = delete all resource types; -all=delete every object of that resource type 
kubectl delete all --all
```

</details>

### Create from terminal

```bash
# Create pod with image name
kubectl run nginx --image=nginx

kubectl run nginx --image=nginx --dry-run=clien -o yaml
```

### Edit pods

Either edit the pod definition.

Or get the pod definition, delete and recreate the pod:
```bash
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```

Or edit the pod's properties with
```bash
kubectl edit pod <pod-name>
```

### Create / delete via configuration file

<details>
    <summary>commands</summary>

```bash
kubectl create -f [file-name.yaml]

kubectl apply -f [file-name.yaml]

kubectl create <something> --dry-run=client -o yaml | kubectl apply -f -

kubectl replace -f [file-name.yaml]

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
kubectl exec -it [pod-name] -- bin/bash

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

## Links

- https://github.com/dgkanatsios/CKAD-exercises/
- https://medium.com/@wely.lau/learning-materials-and-8-tips-to-pass-ckad-certified-kubernetes-application-developer-exam-433499086f3b

- https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-config/
- https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/
- https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/

- https://github.com/jetstack/kubernetes.github.io/blob/master/docs/concepts/configuration/secret.md
- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
