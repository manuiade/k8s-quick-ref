# Troubleshooting


## Cluster troubleshooting

Backup kube-apiserver manifest and deploy a broken version:

```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/kube-apiserver.yaml.ori
# add a broken argument and check container restart
watch crictl ps
kubectl get pods -n kube-system
# This will not work
```

Check kube-apiserver container logs directly from known log locations:

```bash
cat /var/log/pods/kube-system_kube-apiserver-controlplane_36b3bac598f6bd76f4b97286d2bcb99b/kube-apiserver/4.log
cat /var/log/containers/kube-apiserver-controlplane_kube-system_kube-apiserver-fe91f06c179b5dfd299a2116a9c4c2c06584908d4f3eeebdba5435ccf48227fb.log
crictl logs <KUBE_APISERVER_CONTAINER_ID>
```

Restore original (kube-apiserver will work again):

```bash
cp ~/kube-apiserver.yaml.ori /etc/kubernetes/manifests/kube-apiserver.yaml
kubectl get pods -n kube-system
```

Add a broken YAML syntax:

```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/kube-apiserver.yaml.ori
# add a broken YAML syntax and check container restart
watch crictl ps
kubectl get pods -n kube-system
# This will not work
```

Note that container logs will contain nothing useful (since container is never restarted):

```bash
cat /var/log/pods/kube-system_kube-apiserver-controlplane_36b3bac598f6bd76f4b97286d2bcb99b/kube-apiserver/4.log
cat /var/log/containers/kube-apiserver-controlplane_kube-system_kube-apiserver-fe91f06c179b5dfd299a2116a9c4c2c06584908d4f3eeebdba5435ccf48227fb.log
crictl logs <KUBE_APISERVER_CONTAINER_ID>
```

Check systemd logs (or journalctl) to capture error events:

```bash
tail -f /var/log/syslog | grep apiserver
journalctl | grep apiserver
```

Worker node kubelet configuration file location for troubleshooting:

```bash
systemctl status kubelet
cat /etc/kubernetes/kubelet.conf
cat /var/lib/kubelet/config.yaml
```

## Troubleshooting issues related to coreDNS

If you find CoreDNS pods in pending state first check network plugin is installed.

If coredns pods have CrashLoopBackOff or Error state, check the logs

If you have nodes that are running SELinux with an older version of Docker you might experience a scenario where the coredns pods are not starting. To solve that you can try one of the following options:

- Upgrade to a newer version of Docker.

- Disable SELinux.

- Modify the coredns deployment to set allowPrivilegeEscalation to true:

```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
```

Another cause for CoreDNS to have CrashLoopBackOff is when a CoreDNS Pod deployed in Kubernetes detects a loop:

- Add the following to your kubelet config yaml: resolvConf: <path-to-your-real-resolv-conf-file> This flag tells kubelet to pass an alternate resolv.conf to Pods. For systems using systemd-resolved, /run/systemd/resolve/resolv.conf is typically the location of the "real" resolv.conf, although this can be different depending on your distribution.

- Disable the local DNS cache on host nodes, and restore /etc/resolv.conf to the original.

- A quick fix is to edit your Corefile, replacing forward . /etc/resolv.conf with the IP address of your upstream DNS, for example forward . 8.8.8.8. But this only fixes the issue for CoreDNS, kubelet will continue to forward the invalid resolv.conf to all default dnsPolicy Pods, leaving them unable to resolve DNS.


If CoreDNS pods and the kube-dns service is working fine, check the kube-dns service has valid endpoints.

```bash
kubectl -n kube-system get ep kube-dns
```

If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports.


## Troubleshooting issues related to Kube Proxy

In a cluster configured with kubeadm, you can find kube-proxy as a daemonset.

kubeproxy is responsible for watching services and endpoint associated with each service. When the client is going to connect to the service using the virtual IP the kubeproxy is responsible for sending traffic to actual pods.

If you run a kubectl describe ds kube-proxy -n kube-system you can see that the kube-proxy binary runs with following command inside the kube-proxy container.

```bash
/usr/local/bin/kube-proxy
    --config=/var/lib/kube-proxy/config.conf
    --hostname-override=$(NODE_NAME)
``` 

So it fetches the configuration from a configuration file ie, /var/lib/kube-proxy/config.conf and we can override the hostname with the node name of at which the pod is running.
 
In the config file we define the clusterCIDR, kubeproxy mode, ipvs, iptables, bindaddress, kube-config etc.

Troubleshooting issues related to kube-proxy:

- Check kube-proxy pod in the kube-system namespace is running.

- Check kube-proxy logs.

- Check configmap is correctly defined and the config file for running kube-proxy binary is correct.

- kube-config is defined in the config map.

- check kube-proxy is running inside the container

```bash
netstat -plan | grep kube-proxy
```

## References

- https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

- https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/