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