# Kubernetes Hello World Webserver with Minikube

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Install Minikube](#install-minikube)
- [Create Docker Image](#create-docker-image)
- [Create Pod in Kubernetes](#create-pod-in-kubernetes)
- [Create NodePort Service](#create-nodeport-service)
- [Access the Service](#access-the-service)

## Introduction
This guide demonstrates how to deploy a simple "Hello, World!" webserver using Kubernetes and Minikube. The webserver is hosted in a Docker container, and a NodePort service is created to make it accessible.

## Prerequisites
- Docker installed: [Docker Installation Guide](https://docs.docker.com/engine/install/)

## Install Minikube
Follow the installation guide for your operating system: 
[Minikube Installation Guide](https://minikube.sigs.k8s.io/docs/start)

Once the installation is done, start Minikube using the following command.

```bash
minikube start
😄  minikube v1.30.1 on Ubuntu 20.04
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🎉  minikube 1.32.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.32.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.26.3 on Docker 23.0.2 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /snap/bin/kubectl is version 1.18.20, which may have incompatibilities with Kubernetes 1.26.3.
    ▪ Want kubectl v1.26.3? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## Create Docker Image
Create a custom Docker image based on the NGINX webserver with a basic "Hello, World!" HTML page.

```bash
cat index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello, World!</title>
</head>
<body>

    <header>
        <h1>Hello, World!</h1>
    </header>

    <main>
        <p>This webserver is running in Kubernetes.</p>
    </main>

    <footer>
        <p>Copyright 2024.</p>
    </footer>

</body>
</html>
```
The Dockerfile

```bash
cat Dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```
Run docker build command

```bash
docker build -t chiteodor/basic-webserver:latest .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM nginx:latest
 ---> 605c77e624dd
Step 2/2 : COPY index.html /usr/share/nginx/html/index.html
 ---> Using cache
 ---> 08de8861d995
Successfully built 08de8861d995
Successfully tagged chiteodor/basic-webserver:latest
```
Push the Docker image to DockerHub for access in Kubernetes.

```bash
docker push chiteodor/basic-webserver:latest
The push refers to repository [docker.io/chiteodor/basic-webserver]
df740f13563f: Pushed 
d874fd2bc83b: Pushed 
32ce5f6a5106: Pushed 
f1db227348d0: Pushed 
b8d6e692a25e: Pushed 
e379e8aedd4d: Pushed 
2edcec3590a4: Pushed 
latest: digest: sha256:dc326d7c69c7f5d7bda0eca05c61e9a59b3a67a304cf9bf41297f85ccb6849b2 size: 1777
```
## Create Pod in Kubernetes
Create a Kubernetes Pod using the previously created Docker image.

```bash
cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: basic-webserver
  labels:
    app: basic-webserver
spec:
  containers:
  - name: basic-webserver
    image: chiteodor/basic-webserver:latest
    ports:
    - containerPort: 80
  restartPolicy: Always
```
Run kubectl apply command

```bash
kubectl apply -f pod.yaml
pod/basic-webserver created
```
Verify the Pod is running:

```bash
kubectl get pod
NAME                  READY   STATUS    RESTARTS   AGE
pod/basic-webserver   1/1     Running   0          89s
```
## Create NodePort Service
Create a NodePort service to expose the webserver on port 30080.

```bash
cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: basic-webserver
spec:
  selector:
    app: basic-webserver
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```
Run kubectl apply command

```bash
kubectl apply -f service.yaml
service/basic-webserver created
```
Verify the Pod is running:

```bash
kubectl get service
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/basic-webserver   NodePort    10.111.112.131   <none>        80:30080/TCP   89s
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        209d
```
## Access the Service
Check the Minikube endpoint to access the service

```bash
minikube service basic-webserver --url
http://192.168.49.2:30080
```


