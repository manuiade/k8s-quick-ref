# Network policies

- https://kubernetes.io/docs/concepts/services-networking/network-policies/

- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

## Demo

```
kubectl apply -f test.yaml
kubectl apply -f ingress-policy.yaml
```

Try connectivity to nginx from allowed and unallowed busybox:

```
# This should work
kubectl exec -it busybox-allowed -- wget -qO- --timeout=5 nginx

# This should NOT work
kubectl exec -it busybox-unallowed -- wget -qO- --timeout=5 nginx
```
