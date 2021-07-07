# Pod Management

- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

- https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

## Security Context

Create a user for testing:

```
sudo useradd -u 2000 allowed-user
sudo groupadd -g 3000 allowed-group
sudo useradd -u 2001 unallowed-user
sudo groupadd -g 3001 unallowed-group
sudo mkdir -p /etc/message/
echo "I have permission to read this!" | sudo tee -a /etc/message/message.txt
sudo chown 2000:3000 /etc/message/message.txt
sudo chmod 640 /etc/message/message.txt
```

## Node selector

Add a label to the desired node and apply the pod:

```
kubectl label nodes <node-name> node-label=selected
kubectl apply -f node-selector.yaml
```

Check if the pod is running on the correct node:
```
kubectl get pods -o wide
```

## Taints and tolerations

Add a taint to the labeled node:

```
kubectl label nodes <node-name> node-label=selected
kubectl taint node -l node-label=selected key1=value1:NoSchedule
# The NoExecute effect will evict pod already running on that node
```

Create the 2 pods and see where they have been placed:
```
kubectl apply -f pod-tolerations.yaml
kubectl get pods -o wide
```

