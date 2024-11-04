# Networking

## Network Namespaces

Create network namespaces:

```bash
ip netns add red
ip netns add blue
```

List network interfaces on host and network namespaces:

```bash
ip link
ip netns exec red ip link
ip netns exec blue ip link
```

Check routing table on host and network namespaces.

```bash
ip route
ip netns exec red ip route
ip netns exec blue ip route
```

Create a virtual ethernet interface (veth or virtual cable) to connect network namespaces:

```bash
ip link add veth-red type veth peer name veth-blue
ip link set veth-red netns red
ip link set veth-blue netns blue
```

Add address to each veth and bring it up:

```bash
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blue
ip -n red link set veth-red up
ip -n blue link set veth-blue up
```

Check connectivity between the 2 veths:

```bash
ip netns exec red ping 192.168.15.2
```

Clear the network namespaces:

```bash
ip -n red link del veth-red
ip netns del red
ip netns del blue
```

Enable communication in overlay network with a virtual switch (linux bridge):

```bash
ip netns add red
ip netns add blue

ip link add v-net-0 type bridge
ip address show type bridge

ip link add veth-red type veth peer name veth-red-br
ip link set veth-red netns red
ip link set veth-red-br master v-net-0
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n red link set veth-red up

ip link add veth-blue type veth peer name veth-blue-br
ip link set veth-blue netns blue
ip link set veth-blue-br master v-net-0
ip -n blue addr add 192.168.15.2/24 dev veth-blue
ip -n blue link set veth-blue up

ip link set dev veth-red-br up
ip link set dev veth-blue-br up
ip link set dev v-net-0 up

ip netns exec red ping 192.168.15.2
```

Reaches the bridge network interface from host by assigning an IP to it:

```bash
ip addr add 192.168.15.5/24 dev v-net-0
ping 192.168.15.2
```

Add default route to send all external traffic to bridge gateway:

```bash
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
ip netns exec blue ip route add default via 192.168.15.5
ip netns exec blue ip route
ip netns exec blue ping 192.168.1.1

# enable nat on the host to forward packets
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
ip netns exec blue ping 8.8.8.8
```

Add DNAT rule to expose network namespace directly from external (port forwarding):

```bash
iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
iptables -nvL -t nat
```

## Calico CNI [WIP]

Run these commands in cluster nodes where Calico CNI plugin is used

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

Check CNI script called by container runtime:

```bash
cat /etc/cni/net.d/net-script.conflist
cat /opt/cin/bin/<CNI>
```

Call NodePort

```bash
kubectl get svc
curl 192.168.178.2:32061
```

Call pod from other pod

```bash
kubectl get pods -o wide
kubectl exec -it nginx-7f456874f4-5xc59 -- /bin/bash
# call other pod IP
curl http://10.244.182.2
```

Capture network packets with tshark (from node with pod making the call)

```bash
tshark -i wlan0 -V -Y "http"
curl http://10.244.182.2
```

Note in the packet there will be 2 IP header nested (the one internal with the tunl0 IP source and dest pod IP)


### Calicoctl

Install calicoctl:

```bash
curl -L https://github.com/projectcalico/calico/releases/latest/download/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
export DATASTORE_TYPE=kubernetes
export KUBECONFIG=./kube/config
```

Get info on IpPool for network configuration (IP-in-IP)

```bash
calicoctl get workloadendpoints --allow-version-mismatch
calicoctl get ipPool
calicoctl get ipPool default-ipv4-ippool -o yaml > ippool.yaml
```

Watch route for modifications:

```bash
watch route -n
```

Change the ipipMode to Never (from Always)

```bash
calicoctl apply -f ippool.yaml
```

Note tunl0 route changing to wlan0

Retry a tshark inspection and note that there is no encapsulation with ip-in-ip

Call pod from other pod:

```bash
kubectl exec -it nginx-7f456874f4-5xc59 -- /bin/bash
tshark -i wlan0 -V -Y "http"
curl http://10.244.182.2
```


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

## Kube Proxy

Check if kube-proxy is using iptables (from each worker node kube-proxy containers are running):

```bash
crictl ps | grep kube-proxy
crictl logs <KUBE_PROXY_CONTAINER_ID> | grep iptables
```

Check iptables rules for a specific k8s service:

```bash
iptables-save | grep <SERVICE>
```


## References

- https://kubernetes.io/docs/concepts/services-networking/service/

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

- https://kubernetes.io/docs/concepts/services-networking/network-policies/

- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

- https://raw.githubusercontent.com/kubernetes/design-proposals-archive/main/network/networking.md

- https://kubernetes.io/docs/reference/networking/virtual-ips/