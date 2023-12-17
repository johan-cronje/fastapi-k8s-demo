# fastapi-k8s-demo
This repo demonstrates how to deploy a Python FastAPI application to a Kubernetes cluster.

## Prerequisites

### Kubernetes
I use [k3d](https://k3d.io/) which is a Docker wrapper around [K3S](https://k3s.io/) which is a Lightweight Kubernetes distribution built for IoT & Edge computing.

Use the command `k3d create cluster` to create a new k3s cluster with containerized nodes (k3s in docker)

Every cluster will consist of one or more containers:
 * 1 (or more) server node container (k3s)
 * (optionally) 1 loadbalancer container as the entrypoint to the cluster (nginx)
 * (optionally) 1 (or more) agent node containers (k3s)

Typical flags:
```
  -a, --agents int
      Specify how many agents you want to create

      --api-port [HOST:]HOSTPORT
      Specify the Kubernetes API server port exposed on the LoadBalancer (Format: [HOST:]HOSTPORT)
       - Example: `k3d cluster create --servers 3 --api-port 0.0.0.0:6550`

  -p, --port [HOST:][HOSTPORT:]CONTAINERPORT[/PROTOCOL][@NODEFILTER]
      Map ports from the node containers (via the serverlb) to the host (Format: [HOST:][HOSTPORT:]CONTAINERPORT[/PROTOCOL][@NODEFILTER])
       - Example: `k3d cluster create --agents 2 -p 8080:80@agent:0 -p 8081@agent:1`

  -s, --servers int
      Specify how many servers you want to create

  -v, --volume [SOURCE:]DEST[@NODEFILTER[;NODEFILTER...]]
      Mount volumes into the nodes (Format: [SOURCE:]DEST[@NODEFILTER[;NODEFILTER...]]
```

```bash
# create a cluster with 1 server, 2 agents and define the listening ports of your Traefik instance
k3d cluster create --agents 2 -p '9080:80@loadbalancer' -p '9443:443@loadbalancer'

# see the list of k3d clusters
k3d cluster list

# list the local Kubernetes contexts
kubectl config get-contexts
# switch cluster context (i.e. which kubectl talks to)
kubectl config use-context k3d-k3s-default

# delete all clusters when done
k3d cluster delete --all
```
