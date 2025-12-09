Check kubernetes components containers are running or not.
```
crictl --runtime-endpoint=unix:///run/containerd/containerd.sock ps -a | egrep 'etcd|apiserver|controller|scheduler'
```

Static pods manifests file:
```
ls /etc/kubernetes/manifests/
```

Force stop all containers:
if error:
```
rm: cannot remove '/var/lib/kubelet/pods/417fa5dc-f33f-4532-903a-f8a5d027d425/volumes/kubernetes.io~projected/kube-api-access-dmmpd': Device or resource busy

```
Then run,
```
# Stop all running containers
crictl --runtime-endpoint=unix:///run/containerd/containerd.sock stop $(crictl --runtime-endpoint=unix:///run/containerd/containerd.sock ps -q) 2>/dev/null

# Remove all containers
crictl --runtime-endpoint=unix:///run/containerd/containerd.sock rm $(crictl --runtime-endpoint=unix:///run/containerd/containerd.sock ps -aq) 2>/dev/null

# Stop all pod sandboxes
crictl --runtime-endpoint=unix:///run/containerd/containerd.sock stopp $(crictl --runtime-endpoint=unix:///run/containerd/containerd.sock pods -q) 2>/dev/null

# Remove all pod sandboxes
crictl --runtime-endpoint=unix:///run/containerd/containerd.sock rmp $(crictl --runtime-endpoint=unix:///run/containerd/containerd.sock pods -q) 2>/dev/null
```
