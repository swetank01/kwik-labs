# Overview
In this lab you will learn how to:

- Provision a complete Kubernetes cluster using Kubernetes Engine.
- Deploy and manage Docker containers using kubectl.
- Break an application into microservices using Kubernetes' Deployments and Services.
- Kubernetes is all about applications. In this part of the lab you will use an example application called "app".

App is hosted on GitHub and provides an example 12-Factor application. During this lab you will be working with the following Docker images:

kelseyhightower/monolith - Monolith includes auth and hello services.
kelseyhightower/auth - Auth microservice. Generates JWT tokens for authenticated users.
kelseyhightower/hello - Hello microservice. Greets authenticated users.
nginx - Frontend to the auth and hello services.

# Setup and Requirements
```
gcloud auth list
gcloud config list project
gcloud config set compute/zone us-central1-b
gcloud container clusters create io
```

# Get the sample code
```
gsutil cp -r gs://spls/gsp021/* .
cd orchestrate-with-kubernetes/kubernetes
ls
```

# Quick Kubernetes Demo
```
kubectl create deployment nginx --image=nginx:1.10.0
kubectl get pods
kubectl expose deployment nginx --port 80 --type LoadBalancer
kubectl get services
curl http://<External IP>:80
```

# Pods
At the core of Kubernetes is the Pod.

Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

![pod](assets/pods_arch.jpeg)

In this example there is a pod that contains the monolith and nginx containers.

Pods also have Volumes. Volumes are data disks that live as long as the pods live, and can be used by the containers in that pod. Pods provide a shared namespace for their contents which means that the two containers inside of our example pod can communicate with each other, and they also share the attached volumes.

Pods also share a network namespace. This means that there is one IP Address per pod.

Next, a deeper dive into pods.

## Creating Pods

Pods can be created using pod configuration files. Take a moment to explore the monolith pod configuration file.

Run the following:

`cat pods/monolith.yaml`

There's a few things to notice here. You'll see that:

- your pod is made up of one container (the monolith).
- you're passing a few arguments to our container when it starts up.
- you're opening up port 80 for http traffic.

Create the monolith pod using kubectl:

`kubectl create -f pods/monolith.yaml`

Examine your pods. Use the kubectl get pods command to list all pods running in the default namespace:

`kubectl get pods`

Note: It may take a few seconds before the monolith pod is up and running. The monolith container image needs to be pulled from the Docker Hub before you can run it.

Once the pod is running, use kubectl describe command to get more information about the monolith pod:

`kubectl describe pods monolith`

You'll see a lot of the information about the monolith pod including the Pod IP address and the event log. This information will come in handy when troubleshooting.

Kubernetes makes it easy to create pods by describing them in configuration files and easy to view information about them when they are running. At this point you have the ability to create all the pods your deployment requires!


# Interacting with Pods

By default, pods are allocated a private IP address and cannot be reached outside of the cluster. Use the kubectl port-forward command to map a local port to a port inside the monolith pod.

* From this point on the lab will ask you to work in multiple cloud shell tabs to set up communication between the pods. Any commands that are executed in a second or third command shell will be denoted in the command's instructions.

Open a second Cloud Shell terminals. One to run the kubectl port-forward command, and the other to issue curl commands.

In the 2nd terminal, run this command to set up port-forwarding:

`kubectl port-forward monolith 10080:80`

Now in the 1st terminal start talking to your pod using *curl*:

`curl http://127.0.0.1:10080`

Yes! You got a very friendly "hello" back from your container.

Now use the curl command to see what happens when you hit a secure endpoint:

`curl http://127.0.0.1:10080/secure`

Try logging in to get an auth token back from the monolith:

`curl -u user http://127.0.0.1:10080/login`

At the login prompt, use the super-secret password "password" to login.

Logging in caused a JWT token to print out. Since Cloud Shell does not handle copying long strings well, create an environment variable for the token.

`TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')`

Enter the super-secret password "password" again when prompted for the host password.Use this command to copy and then use the token to hit the secure endpoint with curl:

`curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure`

At this point you should get a response back from our application, letting us know everything is right in the world again. Use the kubectl logs command to view the logs for the monolith Pod.

`kubectl logs monolith`

Open a 3rd terminal and use the -f flag to get a stream of the logs happening in real-time:

`kubectl logs -f monolith`

Now if you use curl in the 1st terminal to interact with the monolith, you can see the logs updating (in the 3rd terminal):

`curl http://127.0.0.1:10080`

Use the kubectl exec command to run an interactive shell inside the Monolith Pod. This can come in handy when you want to troubleshoot from within a container:

`kubectl exec monolith --stdin --tty -c monolith /bin/sh`

For example, once you have a shell into the monolith container you can test external connectivity using the ping command:

`ping -c 3 google.com`

Be sure to log out when you're done with this interactive shell.

`exit`

As you can see, interacting with pods is as easy as using the kubectl command. If you need to hit a container remotely, or get a login shell, Kubernetes provides everything you need to get up and going.

# Services

Pods aren't meant to be persistent. They can be stopped or started for many reasons - like failed liveness or readiness checks - and this leads to a problem:

What happens if you want to communicate with a set of Pods? When they get restarted they might have a different IP address.

That's where Services come in. Services provide stable endpoints for Pods.

![pod](assets/service.jpeg)


Services use labels to determine what Pods they operate on. If Pods have the correct labels, they are automatically picked up and exposed by our services.

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:

- ClusterIP (internal) -- the default type means that this Service is only visible inside of the cluster.
- NodePort gives each node in the cluster an externally accessible IP.
- LoadBalancer adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

Now you'll learn how to:

Create a service

Use label selectors to expose a limited set of Pods externally

## Creating a Service

Before you can create our services, first create a secure pod that can handle https traffic.

If you've changed directories, make sure you return to the ~/orchestrate-with-kubernetes/kubernetes directory:

`cd ~/orchestrate-with-kubernetes/kubernetes`

Explore the monolith service configuration file:

`cat pods/secure-monolith.yaml`

Create the secure-monolith pods and their configuration data:

```
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

Now that you have a secure pod, it's time to expose the secure-monolith Pod externally.To do that, create a Kubernetes service.

Explore the monolith service configuration file:

`cat services/monolith.yaml`

Things to note:

1. There's a selector which is used to automatically find and expose any pods with the labels *app: monolith* and *secure: enabled*.

2. Now you have to expose the nodeport here because this is how you'll forward external traffic from port 31000 to nginx (on port 443).

Use the `kubectl create` command to create the monolith service from the monolith service configuration file:

`kubectl create -f services/monolith.yaml`

You're using a port to expose the service. This means that it's possible to have port collisions if another app tries to bind to port 31000 on one of your servers.

Normally, Kubernetes would handle this port assignment. In this lab you chose a port so that it's easier to configure health checks later on.

Use the gcloud compute firewall-rules command to allow traffic to the monolith service on the exposed nodeport:

```
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```

Now that everything is setup you should be able to hit the secure-monolith service from outside the cluster without using port forwarding.

First, get an external IP address for one of the nodes.

`gcloud compute instances list`

Now try hitting the secure-monolith service using curl:

curl -k https://<EXTERNAL_IP>:31000
