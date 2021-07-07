# RBAC

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

- https://kubernetes.io/docs/reference/access-authn-authz/rbac/

## Roles

Create users for testing:
```
sudo useradd allowed-user
sudo passwd allowed-user
sudo useradd unallowed-user
sudo passwd unallowed-user
```

Deploy pods in different namespaces:
```
kubectl apply -f test.yaml
```

Activate the role and clusterrole and verify pod listing from the 2 users created:
```
kubectl apply -f role-binding.yaml
kubectl apply -f cluster-role-binding.yaml
```