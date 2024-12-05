# Cluster Maintenance

## OS upgrades and Patching of cluster

- We can drain the node on K8s cluster for which we want to patch
- this will make the node unschedulable and whatever pods present on the node will be terminated and created on some other node which part of the replica set.
- if the pods on the node which is supposed tobe drain is not a part of replicaset of replication controller or deamonSet then that node will note be able to drain because that pod will not created on other node. So for this we need to use the `--force` option.

**Command**

```
kubectl drain `node_name`
kubectl drain controlplane --ignore-daemonsets
kubectl drain controlplane --force --ignore-daemonsets # when the pod is present and not a part of replicaSet
```

- with drain you can **empty** node and make the node **unschedulable**
- after the pathing is done you can make the node normal by making it uncorden.

**Command**

```
kubectl uncordon `node_name`
```

- this will make the node schedulable. and the new pods will be spawned when the pods on the other node will be terminated or crashed.

**Command**

```
kubectl cordon 'node_name'
```

- This will cordon the node means this make the node unshedulable, But the pods which are already presnet on the node will not be evicted.
- **Taints** offer more granular control compared to cordoning

## Cluster Upgrade Process

- always the `kube-apiserver` will be one or two versions higher than other cluster components
- but the `kubectl` can be higher than `kube-apiserver`
- ![alt text](image-8.png)
- whenever the new version of k8s is releases then the `X-3` version will be unsupported.
- recommended way of upgrading is upgrading minor versios
  - ex - `1.10 --> 1.11 --> 1.12 --> 1.13`
- you can easily upgrade the k8s version on managed k8s on cloud provider with just few clicks
- for `kubeadm` you can use the
  - `kubeadm upgrade plan`
  - `kubeadm upgrade apply`
    - in kubeadm upgrade the kublets are not upgraded you have to manually upgrade the kubelets
    - when you do `kubectl get node` it will only shows the versions of the kubelet on the master and worker nodes
- if you have deployed the k8s cluster from scratch then you need to upgrade each compomnents manually.

### Cluster upgrade strategy

- **Strategy-1**
  - upgrade the mastet node first this will not give downtime as the pods in the nodes are still serving the traffic
    - as the master is still upgrading the new pods on the Worker nodes will not be created as a part of `replicaSet`
  - upgrading all node at once this will have a downtime
- **Strategy-2**
  - Upgarding the master node fist this will not give downtime as the pods in the nodes are still serving the traffi
    - as the master is still upgrading the new pods on the Worker nodes will not be created as a part of `replicaSet`
  - Upgarding the node one by one, when a node is been taken down for upgrade the pods on the node will be creted on the different node which is running.

---

# Backup and Restore Methods

- **etcd** is the key-value store for kubernetes cluster
- **There are two way you can backup the cluster configurations**
  - querying the `kube-apiserver`
  - taking a snapshot.db of `etcd`
- You cannot backup the etcd in **managed** K8s cluster

## Backup `ETCD`

- `etcd` stores the state of the cluster
- rather than taking a backup of each individual resource, best way to take the backup of `ETCD database`
- for etcd the data is stored in the `data-dir=/var/lib/etcd`
- etcd comes with a built in snapshot solution
- to take the `etcd` backup first stop the kube-apiserver using `Service kube-apiserver stopped`

```
ETCDCTL_API=3 etcdctl \
    snapshot save snapshot.db
```

- to check the snapshot status

```
ETCDCTL_API=3 etcdctl \
    snapshot status snapshot.db
```

```
ETCDCTL_API=3 etcdctl \
    snapshot restore snapshot.db \
        --data-dir /var/lib/etcd-from-backup
```

- `systemctl daemon-reload`
- `service etcd restart`
- for authentication remember to provide the certs for `etcd`

```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db
```

- if are using a managed k8s you wont be able to access or backup the `etcd`
- best way to query the `api-server`
- to query the `api-server` use the below command

```
kubectl get all --all-namespaces -o yaml > all-deploy-services.yml
```

### what is a `etcdctl`?

- `etcdctl` is a command-line tool for interacting with etcd, a distributed key-value store.
- `etcdctl` is used to:
  - Put, get, and delete keys from etcd
  - Watch for changes to keys
  - Manage etcd cluster membership
  - Perform maintenance tasks, such as defragmentation and compaction

```
etcdctl put <key> <value>
etcdctl get <key>
etcdctl del <key>
# Watches for changes to a key in etcd and prints the new value when it changes.
etcdctl watch <key>
# Manages etcd cluster membership
etcdctl member <subcommand>
```
