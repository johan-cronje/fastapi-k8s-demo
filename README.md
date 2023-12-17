# fastapi-k8s-demo
This repo demonstrates how to deploy a Python FastAPI application to a Kubernetes cluster.

## Prerequisites

### Kubernetes
I use [k3d](https://k3d.io/) which is a Docker wrapper around [K3S](https://k3s.io/), a Lightweight Kubernetes distribution built for IoT & Edge computing.

Every k3d cluster will consist of one or more containers:
 * 1 (or more) server node container (k3s)
 * (optionally) 1 loadbalancer container as the entrypoint to the cluster (nginx)
 * (optionally) 1 (or more) agent node containers (k3s)

By default, k3d will update your default kubeconfig with your new cluster’s details and set the current-context to it.

Typically the configuration for the cluster is stored in a k3d SimpleConfig file [YAML file](k3d/demo_config.yaml). The options can be reviewed in the k3d documentation: [Using Config Files](https://k3d.io/v5.6.0/usage/configfile/)

```bash
# first create the image registry
k3d registry create demo-registry.localhost --port 12345

# create a cluster with 1 server, 2 agents and define the listening ports of your Traefik instance
k3d cluster create --config k3d/demo_config.yaml --registry-use k3d-demo-registry.localhost:12345

# display the ip addresses of the Docker containers that make up the K3d cluster
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q) | grep 'k3d-demo'
# /k3d-demo-serverlb - 192.168.16.5
# /k3d-demo-agent-1 - 192.168.16.3
# /k3d-demo-agent-0 - 192.168.16.4
# /k3d-demo-server-0 - 192.168.16.2

# see cluster info
kubectl cluster-info
# Kubernetes control plane is running at https://0.0.0.0:46317
# CoreDNS is running at https://0.0.0.0:46317/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
# Metrics-server is running at https://0.0.0.0:46317/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

# delete all clusters when done
k3d cluster delete demo

# delete registry
k3d registry delete k3d-demo-registry.localhost
```

Install Dashboard:
```bash
# install Kubernetes Dashboard
kubectl apply -f dashboard/k8s_dashboard.yaml

# verify that Dashboard is deployed and running
kubectl get pod -n kubernetes-dashboard

# create a ServiceAccount and ClusterRoleBinding to provide admin access to the newly created cluster
kubectl apply -f dashboard/dashboard-admin-user.yaml -f dashboard/dashboard-admin-user-role.yml

# create the token to log in to the Dashboard
token=$(kubectl -n kubernetes-dashboard create token admin-user) && echo ${token}

# ingress route to dashboard ???



# you can access your Dashboard using the kubectl command-line tool by running the following command
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'
# OR
kubectl --address='0.0.0.0' -n kubernetes-dashboard port-forward svc/kubernetes-dashboard 9090:80
```
