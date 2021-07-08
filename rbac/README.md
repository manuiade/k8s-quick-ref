# RBAC

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

- https://kubernetes.io/docs/reference/access-authn-authz/rbac/

## Roles

Create service accounts for testing:
```
kubectl apply -f service-account.yaml
```

Deploy pods in different namespaces:
```
kubectl apply -f test.yaml
```

Activate the role:
```
kubectl apply -f role-binding.yaml
```

Verify pod listing as serviceaccount:
```
# This should work
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac-ns:allowed-sa

# This should not work
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac:unallowed-sa

# This should not work
kubectl get pods -n default --as=system:serviceaccount:rbac-ns:allowed-sa
```

Activate the clusterrole:
```
kubectl apply -f cluster-role-binding.yaml
```

Now every service account should have get access to all namespaces:
```
# All should work
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac-ns:allowed-sa
kubectl get pods -n rbac-ns --as=system:serviceaccount:rbac-ns:unallowed-sa
kubectl get pods -n default --as=system:serviceaccount:rbac-ns:allowed-sa
kubectl get pods -n default --as=system:serviceaccount:rbac-ns:unallowed-sa
```