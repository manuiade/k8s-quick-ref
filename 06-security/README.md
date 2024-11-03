# Security

## Authentication

Impersonification examples (user and group):

```bash
kubectl get nodes --as=superman --as-group=system:masters
```

## TLS Certificate Setup

Generate self-signed certificate for root CA

```bash
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

Create a client certificate for an admin user (system:masters group already has admin privileges on cluster):

```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

Make an API call to k8s using client certificates:

```bash
curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt
```

## k8s Certificate APIs

End-to-end certificate signing process

```bash
# Generate a private key
openssl genrsa -out user.key 2048

# Create CSR configuration file
cat <<EOF > user.csr.cnf
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
CN = user              
O = developers #group membership
[ v3_ext ]                                
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE                
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=clientAuth
EOF

# Generate the CSR
openssl req -config user.csr.cnf -new -key user.key -nodes -out user.csr

# Create a CRS object to send to k8s APIs
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user
spec:
  groups:
  - kubeadm:cluster-admins
  request: $(cat user.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

# Approve the CSR request
kubectl certificate approve user

# Download the certificate
kubectl get csr user -o jsonpath='{.status.certificate}' | base64 --decode > user.crt

# Check certificate validity
openssl x509 -in user.crt -noout -text

# Call k8s APIs specifying certificate options with kubectl
kubectl get pods \
    --server <IP>:6443 \
    --client-key admin.key \
    --client-certificate admin.crt \
    --certificate-authority ca.crt
```

## RBAC

Create service accounts for testing:

```bash
kubectl apply -f rbac/service-account.yaml
```

Deploy pods in different namespaces:

```bash
kubectl apply -f rbac/test.yaml
```

Activate the role:

```bash
kubectl apply -f rbac/role-binding.yaml
```

Verify pod listing as serviceaccount (impersonation):

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
kubectl apply -f rbac/cluster-role-binding.yaml
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

## Service Accounts

Create service account from CLI

```bash
kubectl create serviceaccount test-sa
```

Create pods with automount and without and check secrets:

```bash
kubectl apply -f service-accounts/pod-secret-sa.yaml
# check if token is present at path /var/run/secrets/kubernetes.io/serviceaccount
```

Get a time-limited API token for service account (before 1.24 it was done automatically for any new service account):

```bash
kubectl create token test-sa --duration 3600s
```

Create a long-lived API token for service account:

```bash
kubectl apply -f service-accounts/long-lived-token.yaml
kubectl get secret test-sa -o yaml
```

## Security Context

Create user and groups for testing:

```bash
sudo useradd -u 2000 allowed-user
sudo groupadd -g 3000 allowed-group
sudo useradd -u 2001 unallowed-user
sudo groupadd -g 3001 unallowed-group
```

Create a privileged file and creates pods with different security contexts mounting them:

```bash
sudo mkdir -p /tmp/message/
echo "I have permission to read this" | sudo tee -a /tmp/message/message.txt
sudo chown 2000:3000 /tmp/message/message.txt
sudo chmod 640 /tmp/message/message.txt

kubectl apply -f pod-management/security-context.yaml
```

Verify file access works for first pod and not for second one

## References

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

- https://kubernetes.io/docs/reference/access-authn-authz/rbac/