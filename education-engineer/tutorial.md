# Kubernetes - Introduction and Deploy

In this tutorial, you will learn about Kubernetes basic concepts and how to deploy an application with it.

# Prerequisites

This tutorial assumes that you are already familiar with basic concepts about:
- Containers
- Docker
- Kubernetes
- Kubectl

If not, read and the follow the steps below before continuing.

Containers are a software technology that packages up code and all its dependencies to assure the application to be portable and executable regardless the bottom infrastructure.

[Docker](https://www.docker.com/) is a tool that allows to create containers and, thus, deploy applications in a sandbox environment. It uses the Docker engine to abstract the platforms heterogeneity and allows containers portability.

Install and enable Docker locally following the [Docker official guide](https://docs.docker.com/get-docker/).

[Kubernetes](https://kubernetes.io) is an orchestration platform that allows to deploy containerized applications. Kubernetes supports multiple container runtimes including Docker, containerd, and custom implementation of the Kubernetes Container Runtime Interface. 

[Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) is a client command-line tool to send commands to the Kubernetes cluster.

For this tutorial, you will use Kubernetes with Docker container runtime and Kubectl to execute operations on the cluster.


# Configure the environment

You can setup the Kubernetes platform in multiple ways, from on-premise installation to public cloud managed environment.

This tutorial will focus on the on-premise installation and setup via Kind.

## Install Kind

[Kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container nodes.

Kind is available via the following package manager or installing from release binaries.

On macOS via [Homebrew](https://brew.sh/):
```
$ brew install kind
```

On Windows via [Chocolatey](https://chocolatey.org/):
```
$ choco install kind
```

On Linux from release binaries:
```
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
```

## Create a Kubernetes cluster

For this tutorial we do not set a name of the cluster, so we refer to the default name (kind). If you want to specify the cluster name, use the option *--name* in the following commands.

Once you have done the Kind installation, create the Kubernetes cluster.
```shell
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Check the presence of the Kubernetes cluster.
```shell
$ kind get clusters
kind
```

You can display endpoint information about the control plane and service in the cluster via kubectl.
```
$ kubectl cluster-info --context kind-kind
Kubernetes master is running at https://127.0.0.1:32772
KubeDNS is running at https://127.0.0.1:32772/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

# Deploy and verify the application

## Configuration

The Kubernetes configuration contains:
- [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) component to inform Kubernetes how we want to deploy the application
- [service](https://kubernetes.io/docs/concepts/services-networking/service/) component to define how to expose the application within the cluster and externally to the cluster.

For the creation of the deployment component, create a file **deployment-app.yaml** and insert the following configuration.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-app
        resources: {}
status: {}
```

For the creation of the service component, create a file **service-app.yaml** and use the following configuration.

```yml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

Some explainations about the configuration  files:
- *web* is the deployment and service name
- *nodePort* indicates the cluster exposes externaly the application as a static port on any nodes' IP
- *port: 8080* means you can reach the application on the port 8080


## Deploy the application

Use the previous configuration files to deploy the application via kubectl.

Apply the deployment configuration
```shell
$ kubectl apply -f deployment-app.yaml
deployment.apps/web created
```

and the service configuration
```shell
$ kubectl apply -f service-app.yaml
service/web created
```

Verify the *web* deployment component and its status.
```shell
kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           13m
```

Verify the *web* service component and its networking information.
```shell
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
web          NodePort    10.96.95.145   <none>        8080:30441/TCP   13m
```

## Use the application

To access and use the application from an external network, expose the service nodePort through port forwarding with the *port-forward* command. It allows to connect an external request to a running service by selecting a matching pod to port forward.

First, retrieve the pod name via kubectl and save it to a environment variable *PODNAME*.

```shell
$ PODNAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
```

If you are on Windows use the following command.
```shell
$ Set-Variable -Name "PODNAME" -Value $(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
```

Then, start the port forwarding with the container to expose the port to the local network.
```
$ kubectl port-forward $PODNAME 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```
Note that the *port-forward* command does not return so you need another shell to execute the test.

Finally, call the application and verify that everything works by accessing *localhost:8080* via command line.
```shell
$ curl localhost:8080
Hello, world!
Version: 1.0.0
Hostname: web-84fb9498c7-gsmjc
```


# Cleanup

Remove the port-forwarding by stopping the *port-forward* command that is always active if not interrupted.

Delete all the pods and services of the cluster.
```shell
$ kubectl delete pod,service --all
pod "web-84fb9498c7-gsmjc" deleted
service "kubernetes" deleted
service "web" deleted
```

Finally, delete the Kubernetes cluster.
```shell
$ kind delete cluster
Deleting cluster "kind" ...
```


# Next Steps

In this tutorial, you deployed an application on Kubernetes. For more information about Kubernetes, how to extend the configurations, and stay up to date with new features and releases, refer to the [Kubernetes official documentation](https://kubernetes.io/docs/home/).

