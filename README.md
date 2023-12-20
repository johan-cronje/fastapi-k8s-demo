# fastapi-k8s-demo
This repo demonstrates how to deploy a Python FastAPI application to a Kubernetes cluster.

## Prerequisites

* docker
* kubectl
* k3d

I use [k3d](https://k3d.io/) which is a Docker wrapper around [K3S](https://k3s.io/), a Lightweight Kubernetes distribution built for IoT & Edge computing.

Every k3d cluster will consist of one or more containers:
 * 1 (or more) server node container (k3s)
 * (optionally) 1 loadbalancer container as the entrypoint to the cluster (nginx)
 * (optionally) 1 (or more) agent node containers (k3s)

By default, k3d will update your default kubeconfig with your new cluster’s details and set the current-context to it.

Typically the configuration for the cluster is stored in a k3d SimpleConfig file [YAML file](k3d/demo_config.yaml). The options can be reviewed in the k3d documentation: [Using Config Files](https://k3d.io/v5.6.0/usage/configfile/)

### Docker image registry

```bash
# first create the image registry
k3d registry create registry.localhost --port 12345

# build docker image, tag & push to local k3d registry
docker build --tag fastapi-app:latest .

# OPTIONAL: run & test the container
docker run --rm -it -p 9080:8000 fastapi-app:latest
curl localhost:9080
# {"message":"Hello from FastAPI"}

# tag and push image
docker tag fastapi-app:latest k3d-registry.localhost:12345/fastapi-app:latest
docker push k3d-registry.localhost:12345/fastapi-app:latest
```

### Kubernetes

Create the cluster and deploy the API
> [!NOTE]
> If the registry already exists and the image has been pushed, start here

```bash
# create a cluster with 1 server, 2 agents
k3d cluster create --config k3d/k3d-config.yaml

# OPTIONAL: display the ip addresses of the Docker containers that make up the K3d cluster
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q) | grep 'k3d-'

# create the FastAPI app deployment
kubectl apply -f k3d/deployment.yaml

# OPTIONAL: view service
kubectl describe service fastapi-service
```

### Test

Access the FastAPI application in your browser at (http://localhost:9080/) to see a welcome message or (http://localhost:9080/docs) for the OpenAPI Swagger style documentation

### Cleanup

```bash
# stop/start cluster
k3d cluster stop demo
k3d cluster start demo

# delete cluster
k3d cluster delete demo

# delete registry
k3d registry delete k3d-demo-registry.localhost
```

### ToDo
1. Add nginx Ingress Controller so multiple services can be exposed
2. Add Kubernetes Dashboard
