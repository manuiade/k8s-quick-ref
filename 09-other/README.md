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

## JSON Path Queries samples

```bash
# Get pod image (first container)
kubectl get pods <POD> -o jsonpath='{.spec.containers[0].image}'

# List numbers greater than a certain amount (each item in the list is > 40)
$[?( @.field > 40 )]

# List fields that contains a string
$.data[?( @.payload == "string" )].field

# List subfields in a list
jpath '$[0,3]'

# Get subset of list (upper-bound excluded)
jpath '$[0:3]'

# Step options (get only odd elements)
jpath '$[0:8:1]'

# Last N element
jpath '$[-1:0]'

# Wildcard sample
jpath $.prizes[?(@.year == 2014)].laureates[*].firstname

# Prettify a kubectl jsonpath query
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'

# More prettified query using loops
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'

# Add custom columns
kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu

# Sort by (PV capacity)
kubectl get pv --sort-by=.spec.capacity.storage -o custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage
```

## References

- https://kubernetes.io/docs/concepts/workloads/controllers/job/

- https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

- https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/