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

kubectl get nodes

kubectl get pod
# Get more information about the mod
kubectl get pod -o wide
# Watch for changes
kubectl get pod --watch

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

### Create deployment and service for Mongo Express

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