# Administration

## Cluster operations

Show k8s API Server Info:

```bash
kubectl cluster-info
```

Check health of cluster components:

```bash
kubectl get componentstatuses
```

List all queryable resources:

```bash
kubectl api-resources
```

List all API versions:

```bash
kubectl api-versions
```

Check etcd configuration if is a static pod (server crt, key..):

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

## Context and quick configuration

Setup kubectl bash autocompletion:

```bash
source <(kubectl completion bash)
```

Shortcut utils:

```bash
alias k=kubectl
export do="-o yaml --dry-run=client"
```

View the current configuration:

```bash
kubectl config current-context
```

View all context configurations:

```bash
kubectl config view
```

Listing all contexts:

```bash
kubectl config get-contexts
```

Switch between contexts:

```bash
kubectl config use-context <CONTEXT_NAME>
```

Setting a default namespace for kubectl commands:

``` bash
kubectl config set-context --current --namespace=NAMESPACE
```

## Nodes operations

Get node info:

```bash
kubectl get nodes -o wide
```

Get node status:

```bash
kubectl describe nodes <NODE>
```

Label a node:

```bash
kubectl label nodes <nodename> key=value
```

Show node labels:

```bash
kubectl get nodes --show-labels=true
```

Stop pod scheduling on the drained node:

```bash
kubectl cordon <nodename>
```

Drain a node (also cordons it and force delete standalone pods not mananged by a controller):

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-local-data=true --force
```

Uncordon a node to make pod be schedulable onto it:

```bash
kubectl uncordon <node name>
```

Taint a node by label:

```bash
kubectl taint node -l key=value key1=value1:NoSchedule
```

Untaint a node:

```bash
kubectl taint node -l key=value key1=value1:NoSchedule -
```

Get all taints for each node:

```bash
kubectl get nodes -o json | jq '.items[].spec.taints'
```

## Leases

Get node leases:

```bash
kubectl get leases -n kube-node-lease
```

Describe leases for a given node (notice renew time updates):

```bash
kubectl describe leases <LEASES> -n kube-node-lease
```

## Etcd

Install etcdctl client:

```bash
ETCD_VER=v3.5.16

# choose either URL
DOWNLOAD_URL=https://github.com/etcd-io/etcd/releases/download

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

rm /tmp/etcd-download-test/etcd
mv /tmp/etcd-download-test/etcdctl /usr/local/bin
mv /tmp/etcd-download-test/etcdutl /usr/local/bin
etcdctl version
```

Check endpoint healthy status:

```bash
etcdctl endpoint health
```

Check number of members that are part of an etcd cluster:

```bash
ETCDCTL_API=3 etcdctl member list \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key \
    --endpoints $ENDPOINT:2379
```

Perform an etcd built-in snapshot:

```bash
ETCDCTL_API=3 etcdctl snapshot save /etc/etcd-snapshot.db \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key \
    --endpoints $ENDPOINT:2379
```


Verify snapshot status:

```bash
etcdutl --write-out=table snapshot status /etc/etcd-snapshot.db
```

Restore snapshot:

```bash
etcdutl --data-dir <data-dir-location> snapshot restore snapshot.db
# Then change the etcd static pod to use the new data-dir and restart pod if it remains in pending state
kubectl delete pod etcd

# if external etcd change restore db ownership and restart unit service with new configuration
# restart the service and delete pods of scheduler and controller manager
```

Check etcd objects from kubernetes pod:

```bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt get / --prefix --keys-only"
```

## Components status

Check kube-api-server configuration:

```bash
# If deployed as a static pod with kubeadm
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# If deployed as systemd service
cat /etc/systemd/system/kube-apiserver.service

# Check as a process
ps aux | grep kube-apiserver
```

Check kube-controller-manager configuration:

```bash
# If deployed as a static pod with kubeadm
cat /etc/kubernetes/manifests/kube-controller-manager.yaml

# If deployed as systemd service
cat /etc/systemd/system/kube-controller-manager.service

# Check as a process
ps aux | grep kube-controller-manager
```

Check kube-scheduler configuration:

```bash
# If deployed as a static pod with kubeadm
cat /etc/kubernetes/manifests/kube-scheduler.yaml

# If deployed as systemd service
cat /etc/systemd/system/kube-scheduler.service

# Check as a process
ps aux | grep kube-scheduler
```


## Cluster upgrade (kubeadm)

Check upgradable k8s versions:

```bash
kubeadm upgrade plan
```

Upgrade process:

```bash
# Drain control plane if it has pods scheduled onto it
kubectl drain controlplane

# Upgrade kubeadm version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*' && \
sudo apt-mark hold kubelet kubectl

# Upgrade control plane
kubeadm upgrade apply v1.29.0
kubectl uncordon controlplane

# Upgrade master kubelet
sudo apt-mark unhold kubelet && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' && \
sudo apt-mark hold kubelet
systemctl daemon-reload
systemctl restart kubelet

# Upgrade worker nodes (run kubectl command from workstation and linux/kubeadm command from current worker node)
kubectl drain node-1

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.x-*' && \
sudo apt-mark hold kubeadm

kubeadm upgrade node

sudo apt-mark unhold kubelet && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' && \
sudo apt-mark hold kubelet

systemctl daemon-reload
systemctl restart kubelet
kubectl uncordon node-1
```

## Extra

Backup all resources config definitions:

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml 
```