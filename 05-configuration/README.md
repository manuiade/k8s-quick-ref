# Configuration

## Basic commands

Create a configmap from a file (default key name is filename):

```bash
kubectl create configmap [CM] --from-file=[KEYNAME]=[FILENAME]
```

Create a configmap from literal values:

```bash
kubectl create configmap [CM] --from-literal=key1=value1 --from-literal=key2=value2
```

Decode a secret:

```bash
kubectl get secret [SECRET] -o jsonpath='{.data}'
```

Create a configmap from an env-file (file contains key=value lines):

```bash
kubectl create configmap [CM] --from-env-file=[FILENAME1] --from-env-file=[FILENAME2]
```

## Generic Secret and configmap

Apply the manifests:

```bash
kubectl apply -f generic/
```

Explore the configuration inside the pods:

```bash
kubectl logs pod-use-env
kubectl logs pod-use-vol
```

## ServiceAccount token Secret

Create the service account and long-lived token:

```bash
kubectl apply -f sa-token/
```

Check secret token value:

```bash
kubectl get secret/build-robot-secret -o yaml
```

Check logs for pods:

```bash
# Should print the access token
kubectl logs pod-automount

# Should not find the token file
kubectl logs pod-noautomount
```

## Docker config Secrets

Create a docker config secret:

```bash
kubectl create secret docker-registry secret-tiger-docker \
  --docker-email=tiger@acme.example \
  --docker-username=tiger \
  --docker-password=pass1234 \
  --docker-server=my-registry.example:5000
```

Retrieve the secret content:

```bash
kubectl get secret secret-tiger-docker -o jsonpath='{.data.*}' | base64 -d
```

Create service account and pod using imagePullSecret:

```bash
kubectl apply -f docker-config/
```

## Basic Authentication Secrets

## SSH Authentication Secrets

## TLS Secrets

## Secret Encryption at Rest

Create a sample secret and get its value:

```bash
kubectl create secret generic supersecret --from-literal key=supersecret
kubectl get secret supersecret -o yaml
```

Get secret data directly from etcd server:

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/supersecret | hexdump -C
```

Check if encryption at rest is enabled in kube-apiserver:

```bash
ps aux | grep kube-apiserver | grep encryption-provider-config
```

Apply encryption configuration creating an encryption key:

```bash
head -c 32 /dev/urandom | base64

# Add to kube-apiserver argument the following: --encryption-provider-config=/path/to/encryption-config.yaml
# Restart kube-apiserver
```

Verify encryption works:

```bash
kubectl create secret generic supersecret2 --from-literal key=supersecret2

ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/supersecret2 | hexdump -C

# If you update an existing secret it will be encrypted
```


## References

- https://kubernetes.io/docs/concepts/configuration/configmap/

- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

- https://kubernetes.io/docs/concepts/configuration/secret/