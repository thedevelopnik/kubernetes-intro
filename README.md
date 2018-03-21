# Introduction to Kubernetes

## Installation

You'll need [Docker](https://docs.docker.com/install/), [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed.

You'll also want to pull down the containers we will run: `$ docker pull nginx:1.13.9-alpine` and `$ docker pull gospotcheck:go-docker-example`.

## Accessing Minikube

Start it up with a `$ minikube start`

Make sure minikube is running using `$ minikube status`

Then access the dashboard via `$ minikube dashboard`

## Deploying an application

Click on `Deploy a Containerized App`.

* name: `webserver`
* image: `nginx:1.13.9-alpine`
* count: 3
* no service
* labels: app:webserver`

Deploy it and let's go look at the running workloads.

## Examining via kubectl

Coming back to the command line, let's do some listing of what we've deployed so far:

`$ kubectl get deployments`

`$ kubectl get replicasets`

`$ kubectl get pods`

And let's get some more detailed descriptions:

`$ kubectl describe pod <pod-id>`

And let's use a label to search:

`$ kubectl get pods -l app=webserver`

`$ kubectl get pods -l app=api`

Now let's get rid of that deployment:

* `$ kubectl delete deployments webserver`

The general pattern for kubectl commands is `kubectl <action> <type> <object>`

## Deploy From kubectl

Let's deploy an application via a yaml file

`$ kubectl create -f webserver.yml`

`$ kubectl get replicasets`

`$ kubectl get pods`

Now we're going to create a service to access the application:

`$ kubectl create -f webserver-svc.yml`

`$ kubectl get service` will tell us what services are running.

`$ kubectl describe service webserver-service` will get us details.

We need the ip address of the cluster:

`$ minikube ip`

And now we can access the service from that ip address, on the port from earlier, in a browser.

## ConfigMaps

First, let's deploy another app, called pingpong.

`$ kubectl apply -f pingpong.yml`

`$ kubectl apply -f pingpong-svc.yml`

We're using `apply` here because that causes Kubernetes to save the original configuration. We could also do `kubernetes create --save-config`. By saving the configuration internally it will be able to apply incremental updates, making it easier to keep our configuration as code.

This application's code sets a default port of `8080`, but then checks to see if there is a preferred port passed in via env variable.

We just ran it with the default port, so let's make sure we can curl it: `$ curl <minikube ip>:<port>/ping`

Now let's create a configuration that Kubernetes will store. A configuration could be used by any number of applications (if, say, you had an app pushing to a queue and another pulling from it).

There is a pingpong-config.yml file we'll use:

`$ kubectl create -f pingpong-config.yml`

Now let's and apply some changes:

`$ kubectl apply -f configured-pingpong.yml`

We can apply the service change, or we can also update in the console. Go into the dashboard, go to Services, click on the pingpong-service, then Edit, and we can change the port to 8000.

And check to make sure it works:

`$ kubectl describe svc pingpong-service` to get the port.

`$ curl <minikube ip>:<port>/ping`

We can see that now it is mapping the port to a container port of 8000, and the app is reading from that environment variable, so the whole chain still works.

## Secrets

This is a silly example, but now let's make that port *secret*.

`$ kubectl create secret generic port --from-literal=PORT=7000`

And we'll go through the same process as above, and see that we're still able to access the service.

There are several ways you can interact with secrets. They can be created as a file, that is referenced as a volume in the container, for example. Secrets are base64 encoded, and can be encrypted at rest in etcd.

## ConfigMaps vs Secrets

Secrets are limited to 1MB in size. They are kept in a temporary file system and only run on nodes that use the secret. They are kept and transmitted in plain text, so SSL should be enabled between the master and worker nodes in production.

ConfigMaps are just straight up more convenient. They can hold entire config files and JSON blobs (like a redis or prometheus configuration).

Also, when secrets change, pods are not automatically updated. This allows you to rotate keys and by default cut off access to potentially compromised pods. Pods do get automatic updates when a ConfigMap changes.
