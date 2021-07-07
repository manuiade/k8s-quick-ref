# K8S Quick References

- https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

- https://jamesdefabia.github.io/docs/reference/

# todo
services


## Kubectl cheat sheet

### Basic pod operations

Setup kubectl bash autocompletion:
```
source <(kubectl completion bash)
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

Get all queryable resources:
```
kubectl api-resources
```

Generate a manifest file without running the pods:
```
kubectl create pod [POD_NAME] --image=nginx --dry-run=client -o yaml > pod.yml
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
kubectl drain <node-name> --ignore-daemonsets --force
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

### Deployments

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
kubectl get secret [SECRET -o jsonpath='{.data}'
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

Setting a default namespace for kubectl commands:
```
kubectl config set-context --current --namespace=NAMESPACE
```

