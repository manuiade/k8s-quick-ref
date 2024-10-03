# Networking

## Services

Apply the deployment:

```bash
kubectl apply -f services/nginx-test.yaml
```

Test the access to your pods using the different services:

```bash
kubectl apply -f services/cluster-ip.yaml
kubectl apply -f services/node-port.yaml
kubectl apply -f services/load-balancer.yaml
```

Check created endpoint slices for services:

```bash
kubectl get endpointslices -l kubernetes.io/service-name=cluster-ip-svc
```

Quickly expose a deployment with CLI:

```bash
kubectl expose deployment/nginx-deployment --port 80
```

## Ingress [TODO]

## Network policies

Creates pod for testing and a sample ingress policy:

```bash
kubectl apply -f network-policy/test.yaml
kubectl apply -f network-policy/ingress-policy.yaml
```

Try connectivity to nginx from allowed and unallowed busybox:

```bash
# This should work
kubectl exec -it busybox-allowed -- wget -qO- --timeout=5 nginx

# This should NOT work
kubectl exec -it busybox-unallowed -- wget -qO- --timeout=5 nginx
```

## CoreDNS [TODO]

## Calico CNI [WIP]

Install utils:

```bash
sudo apt install tshark
curl -L https://github.com/projectcalico/calico/releases/download/v3.28.1/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
```

Show created veths:

```bash
ip link show type veth
```

Show IP-in-IP (tunl0 on node pod subnet):

```bash
ip link show type ipip
```

Explore routes (note tunl0 routes for cross-node communication):

```bash
ip route
# same subnet same node using directly the veth on the same node
ip route get 10.244.219.68
# different subnet, different node with the ip-in-ip tunnel
ip route get 10.244.235.130
```

Call NodePort

```
kubectl get svc
curl 192.168.178.2:32061
```

### Call pod from other pod

```
kubectl get pods -o wide
kubectl exec -it nginx-7f456874f4-5xc59 -- /bin/bash
# call other pod IP
curl http://10.244.182.2
```

### Capture network packets with tshark (from node with pod making the call)

```
tshark -i wlan0 -V -Y "http"
curl http://10.244.182.2
```

### Note in the packet there will be 2 IP header nested (the one internal with the tunl0 IP source and dest pod IP)


## Calicoctl
```
curl -L https://github.com/projectcalico/calico/releases/latest/download/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
export DATASTORE_TYPE=kubernetes
export KUBECONFIG=./kube/config
```

### Get info on IpPool for network configuration (IP-in-IP)
```
calicoctl get workloadendpoints --allow-version-mismatch
calicoctl get ipPool
calicoctl get ipPool default-ipv4-ippool -o yaml > ippool.yaml
```

### Watch route for modifications
```
watch route -n
```

### Change the ipipMode to Never (from Always)

```
calicoctl apply -f ippool.yaml
```

### Note tunl0 route changing to wlan0


### Retry a tshark inspection and note that there is no encapsulation with ip-in-ip

### Call pod from other pod

```
kubectl exec -it nginx-7f456874f4-5xc59 -- /bin/bash
tshark -i wlan0 -V -Y "http"
curl http://10.244.182.2
```

## References

- https://kubernetes.io/docs/concepts/services-networking/service/

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

- https://kubernetes.io/docs/concepts/services-networking/network-policies/

- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

- https://raw.githubusercontent.com/kubernetes/design-proposals-archive/main/network/networking.md

- https://kubernetes.io/docs/reference/networking/virtual-ips/              