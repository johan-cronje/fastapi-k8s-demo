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

# OPTIONAL: test container
docker run --rm -it -p 9080:8000 fastapi-app:latest
# should be acessible at http://localhost:9080

# tag and push image
docker tag fastapi-app:latest k3d-registry.localhost:12345/fastapi-app:latest
docker push k3d-registry.localhost:12345/fastapi-app:latest
```

### Kubernetes

Create the cluster and deploy the API
> [!NOTE]
> If the registry already exists and the image has been pushed, start here

```bash
# create a cluster with 1 server, 2 agents and define the listening ports of your Traefik instance
k3d cluster create --config k3d/cluster.yaml --registry-use k3d-registry.localhost:12345 --registry-config k3d/registry.yaml

# OPTIONAL: display the ip addresses of the Docker containers that make up the K3d cluster
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q) | grep 'k3d-'

# create the FastAPI app deployment
kubectl apply -f k3d/deployment.yaml

# OPTIONAL: view service
kubectl describe service fastapi-app
```

### Kubernetes Dashboard

```bash
# install Kubernetes Dashboard
kubectl apply -f dashboard/k8s_dashboard.yaml

# verify that Dashboard is deployed and running
kubectl get pod -n kubernetes-dashboard

# create a ServiceAccount and ClusterRoleBinding to provide admin access to the newly created cluster
kubectl apply -f dashboard/dashboard-admin-user.yaml -f dashboard/dashboard-admin-user-role.yml

# create the token to log in to the Dashboard
token=$(kubectl -n kubernetes-dashboard create token admin-user) && echo ${token}

# apply Traefik ingress controller to route the traffic from the incoming request to the kubernetes-dashboard service
kubectl apply -f dashboard/ingress.yaml
```

### Cleanup

```bash
# delete cluster
k3d cluster delete demo

# delete registry
k3d registry delete k3d-demo-registry.localhost
```
