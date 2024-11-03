# Other

## Jobs and Cronjobs

Create Job:

```bash
kubectl apply -f jobs/job.yaml
```

Get running pods generated

```bash
kubect get pods
```

Create Cronjob

```bash
kubectl apply -f jobs/cronjob.yaml
```

## References

- https://kubernetes.io/docs/concepts/workloads/controllers/job/

- https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

- https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/