# Services

- https://kubernetes.io/docs/concepts/services-networking/service/

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

## DNS Services

Create the pods:
```
kubectl apply -f dns-service.yaml
```

Ping from one pod to another
```
kubectl exec -it nginx-1 -- /bin/bash
ping dns-demo-2.dns-demo.default.svc.cluster.local
```

## Services

Apply the deployment:
```
kubectl apply -f nginx-test.yaml
```

Test the access to your pods using the different services:
```
kubectl apply -f cluster-ip.yaml
kubectl apply -f node-port.yaml
kubectl apply -f load-balancer.yaml
```