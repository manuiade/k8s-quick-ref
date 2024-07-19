# K8S Quick Reference

Just a list of useful commands used for managing your k8s clusters. Also some basic yaml descriptors are provided in the respective folders with some lifecycle examples.

This is obviously not intended to be an exhaustive reference or replace any documentation, it is just a quick way to spin up and manage some k8s components for testing and learning. Please refer to the official documentation (https://kubernetes.io/docs/home/) for further deepening.


## Kubectl cheat sheet

### Cluster operations

Show k8s API Server Info:
```
kubectl cluster-info
```

Check health of cluster components:
```
kubectl get componentstatuses
```

List all queryable resources:
```
kubectl api-resources
```

List all API versions:
```
kubectl api-versions
```

Setup kubectl bash autocompletion:
```
source <(kubectl completion bash)
```

### Configuration and context

View the current configuration:
```
kubectl config current-context
```

View all context configurations:
```
kubectl config view
```

Listing all contexts:
```
kubectl config get-contexts
```

Switch between contexts:
```
kubectl config use-context <CONTEXT_NAME>
```

Setting a default namespace for kubectl commands:
```
kubectl config set-context --current --namespace=NAMESPACE
```


### Nodes

Get node info:
```
kubectl get nodes -o wide
```

Label a node:
```
kubectl label nodes <nodename> key=value
```

Drain a node:
```
kubectl drain <node-name> --ignore-daemonsets --delete-local-data=true --force
```

Stop pod scheduling on the drained node:
```
kubectl cordon <nodename>
```

Undrain a node:
```
kubectl uncordon <node name>
```

Taint a node by label:
```
kubectl taint node -l key=value key1=value1:NoSchedule
```

Untaint a node:
```
kubectl taint node -l key=value key1=value1:NoSchedule -
```

Get all taints for each node:
```
kubectl get nodes -o json | jq '.items[].spec.taints'
```





### Configmaps and secrets

Create a configmap from a file:
```
kubectl create configmap [CM] --from-file=[FILENAME]
```

Create a configmap from literal values:
```
kubectl create configmap [CM] --from-literal=key1=value1 --from-literal=key2=value2
```

Decode a secret:
```
kubectl get secret [SECRET] -o jsonpath='{.data}'
```

### Create resources from kubectl

Create namespace:
```
kubectl create namespace <namespace>
```

Create service account:
```
kubectl create sa <serviceaccountname>
```

### DNS Debugging

Start a DNS pod:
```
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

Running nslookup:
```
kubectl exec -i -t dnsutils -- nslookup <dns-entry>
```

Check if coredns is running on cluster:
```
kubectl get pods --namespace=kube-system -l k8s-app=kube-dns
```

Verify if DNS endpoints are exposed:
```
kubectl get endpoints kube-dns --namespace=kube-system
```

### Avanced commands


*Work in progress*


## Other references

- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

- https://jamesdefabia.github.io/docs/reference/
