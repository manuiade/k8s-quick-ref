
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
kubectl run [POD_NAME] --image=nginx --dry-run=client -o yaml > pod.yml
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

Get top pods based on CPU utilization (needs Metrics API):

```bash
kubectl top pod --sort-by='cpu'
```

## Pod Debug with ephemeral containers

Create sample pod:

```bash
kubectl run ephemeral-demo --image=nginx --restart=Never
```

Add a debugging container:

```bash
kubectl debug -it ephemeral-demo --image=busybox:1.28 --target=ephemeral-demo
```

Check pod state to see attached ephemeral container:

```bash
kubectl describe pod ephemeral-demo
```

## Pod Lifecycle

Test pod lifecycle samples:

```bash
kubectl apply -f pod-lifecycle/
```

Delete a pod overriding the default (30s) grace period seconds:

```bash
kubectl delete pod [POD] --grace-period=0
```

## Security Context

Create user and groups for testing:

```bash
sudo useradd -u 2000 allowed-user
sudo groupadd -g 3000 allowed-group
sudo useradd -u 2001 unallowed-user
sudo groupadd -g 3001 unallowed-group
```

Create a privileged file and creates pods with different security contexts mounting them:

```bash
sudo mkdir -p /etc/message/
echo "I have permission to read this!" | sudo tee -a /etc/message/message.txt
sudo chown 2000:3000 /etc/message/message.txt
sudo chmod 640 /etc/message/message.txt

kubectl apply -f pod-management/security-context.yaml
```

Verify file access works for first pod and not for second one

## Node selector

Add a label to the desired node and apply the pod:

```bash
kubectl label nodes <node-name> node-label=selected
kubectl apply -f pod-management/node-selector.yaml
```

Check if the pod is running on the correct node:

```bash
kubectl get pods -o wide
```

## Taints and tolerations

Add a taint to the labeled node:

```bash
kubectl label nodes <node-name> node-label=selected
kubectl taint node -l node-label=selected key1=value1:NoSchedule
# The NoExecute effect will evict pod already running on that node
```

Create the 2 pods and see where they have been placed:

```bash
kubectl apply -f pod-management/pod-tolerations.yaml
kubectl get pods -o wide
```

## References

- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes

- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

- https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/