
## Cluster Orchestration
---

### Create the Cluster
`kind create cluster --name poc --config cluster.yaml`

### Install CNI Plugin. Please wait about 5mins for the nodes to be ready.
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

### Label the Nodes
```bash
kubectl label nodes poc-worker node-role.kubernetes.io/worker=
kubectl label nodes poc-worker2 node-role.kubernetes.io/worker2=
```

### Install Ingress Ngnix Controller
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

### Install Metrics Server
```bash
# deploy metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# run the below to skip verifying CA of serving certificates
kubectl -n kube-system patch deploy metrics-server --type json -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args", "value": [
  "--cert-dir=/tmp",
  "--secure-port=4443",
  "--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname",
  "--kubelet-use-node-status-port",
  "--metric-resolution=15s",
  "--kubelet-insecure-tls"
]}]'
```

## Deploying Applications and Defining Communications
---

### Create Frontend and Backend Apps in the "default" Namespace
```bash
kubectl apply -f resources/frontend-svc-po.yaml
kubectl apply -f resources/backend-svc-po.yaml
```

### Perform a Request from either App
```bash
kubectl -n default exec frontend -- curl --trace - --trace-time backend
kubectl -n default exec backend -- curl --trace - --trace-time frontend
```

### Restrict All Traffic Flow on the default Namespace
`kubectl apply -f policies/ns-default-deny.yaml`

### Allow Ingress and Egress Traffic from Frontend to Backend
`kubectl apply -f policies/po-frontend-egr-ing.yam`