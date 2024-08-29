# Networking

## Services

Apply the deployment:
```
kubectl apply -f nginx-test.yaml
```

Test the access to your pods using the different services:
```
kubectl apply -f services/cluster-ip.yaml
kubectl apply -f services/node-port.yaml
kubectl apply -f services/load-balancer.yaml
```

## Ingress

## CoreDNS

## CNI

## References

- https://kubernetes.io/docs/concepts/services-networking/service/

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/