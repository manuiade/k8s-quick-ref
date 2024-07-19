# Configuration

- https://kubernetes.io/docs/concepts/configuration/configmap/

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

- https://kubernetes.io/docs/concepts/configuration/secret/

## Demo

Apply the manifests:
```
kubectl apply -f .
```

Explore the configuration inside the pods:
```
kubectl logs pod-use-env
kubectl logs pod-use-vol
```