# K8S Quick References

- https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

- https://jamesdefabia.github.io/docs/reference/

# todo
services


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


### Basic pod operations

Validate a yaml syntax:
```
kubectl create --dry-run --validate -f <file>.yaml
```

Create a new k8s object from a manifest file:
```
kubectl apply -f [MANIFEST.YAML]
```

Get all current resources in a given namespace:
```
kubectl get pods -n [NAMESPACE] -o wide
```

Get info on a specific object in yaml format:
```
kubectl get pods <pod-name> -o yaml
```

Generate a manifest file without running the pods:
```
kubectl create pod [POD_NAME] --image=nginx --dry-run=client -o yaml > pod.yml
```

Start a temporary pod which dies on exit:
```
kubectl run --rm -it --image=alpine alpine -- sh
```

Delete a pod (or every other k8s object):
```
kubectl delete pod <podname>
```

Get environment variables inside a Pod:
```
kubectl exec <podname> -- env
```

Copy from host to pod (and viceversa):
```
kubectl cp /local/file <podname>:/pod/path
```

Get resources sorted by name:
```
kubectl get pods --sort-by=.metadata.name
```


### Pods exploration

Enter in a pod and launch a cloud shell:
```
kubectl exec -it [POD-NAME] -- /bin/bash
```

Port forward to a pod:
```
kubectl port-forward [POD-NAME] [LOCALHOST_PORT]:[CONTAINER_PORT]
```

Show logs for given pod:
```
kubectl logs [POD-NAME]
```

Show logs in realtime:
```
kubectl logs <podname> -f
```

Get top pods based on CPU utilization:
```
kubectl top pod --sort-by='cpu'
```

### Deployments

Create a nginx deployment:
```
kubectl create deployment nginx --image=nginx  --replicas=3
```

Expose a deployment using a service:
```
kubectl expose deployment nginx --port=80 --type=NodePort
```

Manually scale deployment:
```
kubectl scale deployment nginx-deployment.yaml -replicas=5
```

Autoscale deployment on cpu-usage
```
kubectl autoscale deployment nginx-deployment --min=3 --max=15 --cpu-percent=75
```

Rolling update a deployment
```
kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
```

Check rolling status
```
kubectl rollout status deployment.v1.apps/nginx-deployment
```

View rolling history
```
kubectl rollout history deployment nginx-deployment
```

View details of a deployment revision
```
kubectl rollout history deployment nginx-deployment --revision=2
```

Rolling back to a previous revision
```
kubectl rollout undo deployment nginx-deployment --to-revision=1
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