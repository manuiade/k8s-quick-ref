# Storage

## Host path


Create a pod writing file to local host path:

```bash
kubectl apply -f host-path.yaml
```

Check ouput locally:

```bash
cat /tmp/data/out.txt
```

## Empty dir

Create a multi-container pod sharing volume via empty dir:

```bash
kubectl apply -f empty-dir.yaml
```

Check output on destination container:

```bash
kubectl logs shared-volume-pod -c busybox2
```

## Storage Classes, PV, PVC

Create Storage Class using local volume provisioner and necessary k8s objects (PV, PVCs) to mount storage to test pod:

```bash
kubectl apply -f storage-class.yaml
kubectl apply -f persistent-volume.yaml
kubectl apply -f persistent-volume-claim.yaml
kubectl apply -f test-pvc.yaml
```

Check output destination:

```bash
cat /tmp/data/message.txt
```

## Stateful sets

Test stateful set:

```bash
kubectl apply -f storage-class.yaml
kubectl apply -f persistent-volume.yaml
kubectl apply -f stateful-set.yaml
```

## References

- https://kubernetes.io/docs/concepts/storage/volumes/

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

- https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/