# Kubernetes Tutorial

Following along with [Kubernetes Tutorial](https://www.youtube.com/watch?v=X48VuDVv0do&t=2s&ab_channel=TechWorldwithNana) from TechWorld with Nana.

<details>
    <summary>Table of Content</summary>

- [Kubernetes Tutorial](#kubernetes-tutorial)
  - [Running locally](#running-locally)
    - [Installation and check of version](#installation-and-check-of-version)
  - [`kubectl` commands](#kubectl-commands)
    - [CRUD commands](#crud-commands)
    - [Status](#status)
- [Get more information about the mod](#get-more-information-about-the-mod)
- [Check status](#check-status)
- [Save status](#save-status)

</details>

---

## Running locally

### Installation and check of version

(For Windows)
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

### Status

<details>
    <summary>commands</summary>

```bash
kubectl get all
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
    <summary>Add the values to the mongodeb-secret.yaml file</summary>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: <base64 encoded>
  mongo-root-password: <base64 encoded>
```

</details>

### Create deployment for mongodb

<details>
    <summary>Reference the secret saved in mongodb-secret.yaml file</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  # ...
  template:
    # ...
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        # Default port
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              # mongodb-secret.yaml > metadata > name
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              # mongodb-secret.yaml > metadata > name
              name: mongodb-secret
              key: mongo-root-password
```

</details>
