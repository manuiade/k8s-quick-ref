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

Create role and clustserrole from kubectl CLI:

```bash
k create role smoke --verb create,delete --resource pods,deployments,sts
k create rolebinding smoke --role smoke --user smoke
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

## RBAC Check

Check whether an acion is allowed for a subject based on RBAC:

```bash
kubectl auth can-i create pods --all-namespace
```

Check to see if service account "foo" of namespace "dev" can list pods in the namespace "prod"
(You must be allowed to use impersonation for the global option "--as"):

```bash
kubectl auth can-i list pods --as=system:serviceaccount:dev:foo -n prod
```

Check to see if I can read pod logs:

```bash
kubectl auth can-i get pods --subresource=log
```


## References

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

- https://kubernetes.io/docs/reference/access-authn-authz/rbac/