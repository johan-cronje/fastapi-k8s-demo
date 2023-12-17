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

Typically the configuration for the cluster is stored in a [YAML file](k3d/demo_config.yaml). The options can be reviewed in the k3d documentation: [Using Config Files](https://k3d.io/v5.6.0/usage/configfile/)

```bash
# create a cluster with 1 server, 2 agents and define the listening ports of your Traefik instance
k3d cluster create --config k3d/demo_config.yaml

# see cluster info
kubectl cluster-info

# see the list of k3d clusters
k3d cluster list

# list the local Kubernetes contexts
kubectl config get-contexts
# switch cluster context (i.e. which kubectl talks to)
kubectl config use-context k3d-demo

# delete all clusters when done
k3d cluster delete demo
```
