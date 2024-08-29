# Deployments

## Basic Commands

Create a nginx deployment:

```bash
kubectl create deployment nginx --image=nginx  --replicas=3
```

Expose a deployment using a service:

```bash
kubectl expose deployment nginx --port=80 --type=NodePort
```

Manually scale deployment:

```bash
kubectl scale deployment nginx-deployment.yaml -replicas=5
```

Autoscale deployment on cpu-usage:

```bash
kubectl autoscale deployment nginx-deployment --min=3 --max=15 --cpu-percent=75
```

Rolling update a deployment:

```bash
kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
```

Check rolling status:

```bash
kubectl rollout status deployment.v1.apps/nginx-deployment
```

View rolling history:

```bash
kubectl rollout history deployment nginx-deployment
```

View details of a deployment revision:

```bash
kubectl rollout history deployment nginx-deployment --revision=2
```

Rolling back to a previous revision:

```bash
kubectl rollout undo deployment nginx-deployment --to-revision=1
```

## Sample

Create deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

Manually scale deployment:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Autoscale deployment on cpu-usage:

```bash
kubectl autoscale deployment nginx-deployment --min=3 --max=15 --cpu-percent=75
```

Rolling update a deployment:

```bash
kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
```

Check rolling status:

```bash
kubectl rollout status deployment.v1.apps/nginx-deployment
```

View rolling history:

```bash
kubectl rollout history deployment nginx-deployment
```

View details of a deployment revision:

```bash
kubectl rollout history deployment nginx-deployment --revision=2
```

Rolling back to a previous revision:

```bash
kubectl rollout undo deployment nginx-deployment --to-revision=1
```

Try the Horizontal Pod Autoscaler:

```bash
kubectl apply -f hpa.yaml
```

Clean:

```bash
kubectl delete -f hpa.yaml
kubectl delete -f nginx-deployment.yaml
```

### Blue-Green Deployment

Deploy workloads:

```bash
kubectl apply -f blue-green/
```

Check pod endpoints:

```bash
kubectl describe svc nginx-service
```

Migrate traffic to the new version:

```bash
kubectl patch service nginx-service -p '{"spec":{"selector":{"version": "v2"}}}'
```

Check changed endpoints:

```bash
kubectl describe svc nginx-service
```

### Canary Deployment

Deploy workloads:

```bash
kubectl apply -f canary/
```

Scale normal deployment to send to canary only small percentage of traffic:

```bash
kubectl scale deployment normal-deployment --replicas=10
```

## References

- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/