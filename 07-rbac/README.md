# RBAC

## Basics

Create service accounts for testing:

```bash
kubectl apply -f service-account.yaml
```

Deploy pods in different namespaces:

```bash
kubectl apply -f test.yaml
```

Activate the role:

```bash
kubectl apply -f role-binding.yaml
```

Verify pod listing as serviceaccount:

```bash
# This should work
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac-ns:allowed-sa

# This should not work
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac:unallowed-sa

# This should not work
kubectl get pods -n default --as=system:serviceaccount:rbac-ns:allowed-sa
```

Activate the clusterrole:

```bash
kubectl apply -f cluster-role-binding.yaml
```

Now every service account should have get access to all namespaces:

```bash
# All should work
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac-ns:allowed-sa
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac-ns:unallowed-sa
kubectl get pods -n default --as=system:serviceaccount:rbac-ns:allowed-sa
kubectl get pods -n default --as=system:serviceaccount:rbac-ns:unallowed-sa
```

## Aggregated ClusterRoles

Create multiple clusterrole with common annotation:

```bash
kubectl apply -f aggregated-clusterroles/clusterroles.yaml
```

Create the aggregated clusterrole (which combines them) and apply binding:

```bash
kubectl apply -f aggregated-clusterroles/aggr-cr.yaml
kubectl apply -f aggregated-clusterroles/clusterrolebinding.yaml
```

Create the a clusterrole automatically aggregated with the edit clusterrole:

```bash
kubectl apply -f aggregated-clusterroles/edit-plus-token.yaml
```

## References

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

- https://kubernetes.io/docs/reference/access-authn-authz/rbac/