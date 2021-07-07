# Volumes

- https://kubernetes.io/docs/concepts/storage/volumes/

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

- https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/

## Host path

```
kubectl apply -f ost-path.yaml
```

Check ouput locally:
```
cat /tmp/data/out.txt
```

## Empty dir

```
kubectl apply -f empty-dir.yaml
```

Check output on destination container:
```
kubectl logs shared-volume-pod -c busybox2
```

## Persistent volumes

```
kubectl apply -f storage-class.yaml
kubectl apply -f persistent-volume.yaml
kubectl apply -f persistent-volume-claim.yaml
kubectl apply -f test-pvc.yaml
```

Check output destination:

```
cat /tmp/data/message.txt
```

## Stateful sets

```
kubectl apply -f storage-class.yaml
kubectl apply -f persistent-volume.yaml
kubectl apply -f stateful-set.yaml
```