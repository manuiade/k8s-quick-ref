
# Pods

## Basic pod operations

Validate a yaml syntax:

```bash
kubectl create --dry-run --validate -f <file>.yaml
```

Create a new k8s object from a manifest file:

```bash
kubectl apply -f [MANIFEST.YAML]
```

Get all current resources in a given namespace:

```bash
kubectl get pods -n [NAMESPACE] -o wide
```

Get info on a specific object in yaml format:

```bash
kubectl get pods <pod-name> -o yaml
```

Generate a manifest file without running the pods:

```bash
kubectl create pod [POD_NAME] --image=nginx --dry-run=client -o yaml > pod.yml
```

Start a temporary pod which dies on exit:

```bash
kubectl run --rm -it --image=alpine alpine -- sh
```

Delete a pod (or every other k8s object):

```bash
kubectl delete pod <podname>
```

Get environment variables inside a Pod:

```bash
kubectl exec <podname> -- env
```

Copy from host to pod (and viceversa):

```bash
kubectl cp /local/file <podname>:/pod/path
```

Get resources sorted by name:

```bash
kubectl get pods --sort-by=.metadata.name
```


## Pods exploration

Enter in a pod and launch a cloud shell:

```bash
kubectl exec -it [POD-NAME] -- /bin/bash
```

Port forward to a pod:

```bash
kubectl port-forward [POD-NAME] [LOCALHOST_PORT]:[CONTAINER_PORT]
```

Show logs for given pod:

```bash
kubectl logs [POD-NAME]
```

Show logs in realtime:

```bash
kubectl logs <podname> -f
```

Get top pods based on CPU utilization:

```bash
kubectl top pod --sort-by='cpu'
```

## Pod Debug with ephemeral containers

Create sample pod:

```bash
kubectl run ephemeral-demo --image=registry.k8s.io/pause:3.1 --restart=Never
```

Add a debugging container:

```bash
kubectl debug -it ephemeral-demo --image=busybox:1.28 --target=ephemeral-demo
```

Check pod state to see attached ephemeral container:

```bash
kubectl describe pod ephemeral-demo
```