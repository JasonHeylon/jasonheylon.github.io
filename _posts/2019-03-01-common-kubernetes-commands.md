---
layout: post
title: "Common Kubernetes Commands"
date: 2019-03-01 00:00:00
categories: deployment
comments: true
---

## Installation
```bash
# install docker
$ brew cask install docker
# instlal kube control commandline
$ brew install kubectl
# install minikube for our laptop
$ brew cask install minikube
# start minikube
$ minikube start
# maybe with other virtual machine driver
$ minikube start --vm-driver xxxx
```

## Image from Dockerfile

```dockerfile
FROM nginx
COPY www/* /usr/share/nginx/html
```
```dash
$ docker build -t my-demo:0.1 .
```

## Create a Pod in k8s
```bash
# we need connect kubenetes docker environment
$ eval $(minikube docker-env)
# to revert previous command
$ eval $(minikube docker-env -u)

$ vi pod.yml
```

```yaml
# pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-demo
  labels:
    app: my-demo
spec:
  containers:
    - name: my-demo
      image: my-demo:0.1
      ports:
        - containerPort: 80
```
```bash
$ kubectl create -f pod.yml
pod "my-demo" created

$ kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
my-demo   1/1       Running   0          4s
```

## Create service in k8s

- We need service to talk with the pod in k8s.

```bash
$ vi service.yml
```

```yml
# service.yml
apiVersion: v1
kind: Service
metadata:
  name: my-demo-service
  labels:
    app: my-demo
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30050
  selector:
    app: my-demo
```

```bash
$ kubectl apply -f pod.yml

$ kubectl create -f service.yml
service "my-demo-svc" created

```

## deployment in k8s

```bash
$ vi deployment.yml
```

```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-demo-deployment
spec:
  replicas: 10
  minReadySeconds: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-demo
    spec:
      containers:
        - name: my-demo-pod
          image: my-demo:0.2
          ports:
            - containerPort: 80

```

```bash
$ kubectl create -f deployment.yml
```

## commands about Pod

```bash
$ kubectl get pods
$ kubectl get pods -all-namespaces
$ kubectl get pods --show-labels
$ kubectl describe pod my-pod

# get pods with label
$ kubectl get pods -l app=first-demo
# add a label to pod
$ kubectl label pods my-pod app=first-demo-version1
# remove a label from pod
$ kubectl label pods my-pod app-
```

## commands about Deployment

```bash
# Scale out
$ kubectl scale --replicas=3 deployment/nginx-app
# online rolling upgrade
$ kubectl rollout app-v1 app-v2 --image=img:v2
# Roll backup
$ kubectl rollout app-v1 app-v2 --rollback
# List rollout
$ kubectl get rs
# Check update status
$ kubectl rollout status deployment/nginx-app
# Check update history
$ kubectl rollout history deployment/nginx-app
# Pause/Resume
$ kubectl rollout pause deployment/nginx-deployment, resume
# Rollback to previous version
$ kubectl rollout undo deployment/nginx-deployment
```

## commands about Service

```bash
# List all services
$ kubectl get services
# List service endpoints
$ kubectl get endpoints
# Get service detail
kubectl get service nginx-service -o yaml
```

## commands about Secrets

```bash
# List secrets
$ kubectl get secrets --all-namespaces
# Create secret from cfg file
$ kubectl create secret generic db-user-pass --from-file./username.txt=
# Generate secret
echo -n 'mypasswd'=, then redirect to =base64 -decode
```

## commands about Volumes

```bash
# List storage class
$ kubectl get storageclass
# Check the mounted volumes
$ kubectl exec storage ls /data
# Check persist volume
$ kubectl describe pv/pv0001
# Copy local file to pod
$ kubectl cp /tmp/my <some-namespace>/<some-pod>:/tmp/server
# Copy pod file to local
$ kubectl cp <some-namespace>/<some-pod>:/tmp/server /tmp/my
```

## commands about Node

```bash
# Mark node as unschedulable
$ kubectl cordon $NDOE_NAME
# Mark node as schedulable
$ kubectl uncordon $NDOE_NAME
# Drain node in preparation for maintenance
$ kubectl drain $NODE_NAME
```

## commands about namespace
```bash
# List authenticated contexts
$ kubectl config get-contexts, ~/.kube/config
# Load context from config file
$ kubectl get cs --kubeconfig kube_config.yml
# Switch context
$ kubectl config use-context <cluster-name>
# Delete the specified context
$ kubectl config delete-context <cluster-name>
# List all namespaces defined
$ kubectl get namespaces
# Set namespace preference
$ kubectl config set-context $(kubectl config current-context) --namespace=<ns1>
# List certificates
$ kubectl get csr
```

## commands about Network

```bash
# Temporarily add a port-forwarding
$ kubectl port-forward redis-izl09 6379
# Add port-forwaring for deployment
$ kubectl port-forward deployment/redis-master 6379:6379
# Add port-forwaring for replicaset
$ kubectl port-forward rs/redis-master 6379:6379
# Add port-forwaring for service
$ kubectl port-forward svc/redis-master 6379:6379
# Get network policy
$ kubectl get NetworkPolicy
```

## related links

- [Kubernetes Examples](https://github.com/kubernetes/examples)
- [cheatsheet-kubernetes-A4](https://github.com/dennyzhang/cheatsheet-kubernetes-A4)
