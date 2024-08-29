# Administration

## Cluster operations

Show k8s API Server Info:

```bash
kubectl cluster-info
```

Check health of cluster components:

```bash
kubectl get componentstatuses
```

List all queryable resources:

```bash
kubectl api-resources
```

List all API versions:

```bash
kubectl api-versions
```

Check etcd configuration (server crt, key..):

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```


## Context and quick configuration

Setup kubectl bash autocompletion:

```bash
source <(kubectl completion bash)
```

Shortcut utils:

```bash
alias k=kubectl
export do="-o yaml --dry-run=client"
```

View the current configuration:

```bash
kubectl config current-context
```

View all context configurations:

```bash
kubectl config view
```

Listing all contexts:

```bash
kubectl config get-contexts
```

Switch between contexts:

```bash
kubectl config use-context <CONTEXT_NAME>
```

Setting a default namespace for kubectl commands:

``` bash
kubectl config set-context --current --namespace=NAMESPACE
```

## Nodes operations

## Nodes

Get node info:

```bash
kubectl get nodes -o wide
```

Label a node:

```bash
kubectl label nodes <nodename> key=value
```

Show node labels:

```bash
kubectl get nodes --show-labels=true
```

Drain a node:

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-local-data=true --force
```

Stop pod scheduling on the drained node:

```bash
kubectl cordon <nodename>
```

Undrain a node:

```bash
kubectl uncordon <node name>
```

Taint a node by label:

```bash
kubectl taint node -l key=value key1=value1:NoSchedule
```

Untaint a node:

```bash
kubectl taint node -l key=value key1=value1:NoSchedule -
```

Get all taints for each node:

```bash
kubectl get nodes -o json | jq '.items[].spec.taints'
```