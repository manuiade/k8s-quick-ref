# Deployments

- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

## Demo

```
kubectl apply -f nginx-deployment.yaml
```

Manually scale deployment
```
kubectl scale deployment nginx-deployment --replicas=5
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

Try the Horizontal Pod Autoscaler:
```
kubectl apply -f hpa.yaml
```

Clean:
```
kubectl delete -f hpa.yaml
kubectl delete -f nginx-deployment.yaml
```

### Blue-Green Deployment

```
kubectl apply -f blue-green/
```

Check pod endpoints:
```
kubectl describe svc nginx-service
```

Migrate traffic to the new version:
```
kubectl patch service nginx-service -p '{"spec":{"selector":{"version": "v2"}}}'
```

Check changed endpoints:
```
kubectl describe svc nginx-service
```

### Canary Deployment

```
kubectl apply -f canary/
```

Scale normal deployment to send to canary only small percentage of traffic

```
kubectl scale deployment normal-deployment --replicas=10
```

