# [Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

- Static Pods are managed directly by the **kubelet daemon** on a specific node, without the API server observing them.
- kube-Scheduler ignore the Static pods
- Static pod are used to depoly the Control plane pods
- static pods are only created by kubelet.
- Static Pods are always bound to one Kubelet on a specific node.
- to Identify the Static pods, they always have a nodeName at the end. ex- **etcd-`controlplane`**, **kube-apiserver-`controlplane`**, **kube-controller-manager-`controlplane`**, **kube-scheduler-`controlplane`** or you can refer to the `OwerReference`in the Pod defination file.
- Kubelet does not manage the Deployment and Replicasset it only manages the Pod
- Static pods are visible to the Kube-Apiserver, but does not control from there
- Kubelet also runs on the master node in Kubernetes to provision the master components as static pods.

**There are two way to craete a Static pod**

1. **Filesystem-hosted static Pod manifest**
2. **Web-hosted static pod manifest**

- Manifests are standard Pod definitions in JSON or YAML format in a specific directory. Use the `staticPodPath: <the directory>` field in the [kubelet configuration file](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/), which periodically scans the directory and creates/deletes static Pods as YAML/JSON files **appear/disappear there**.
- to modify the kubelet use the [`KubeletConfiguration`](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/) YAML file which is the recommended way.

```yml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: "192.168.0.8"
port: 20250
serializeImagePulls: false
evictionHard:
    memory.available:  "100Mi"
    nodefs.available:  "10%"
    nodefs.inodesFree: "5%"
    imagefs.available: "15%"
    staticPodPath: "/etc/kubernetes/manifests"
```

to apply this file

`kubectl apply -f kubelet-config.yaml`

- `staticPodPath: "/etc/kubernetes/manifests"` is were the static-pod manifest are stored this path is not same always _refer the kubelet config file for the mention path_

  - `cat /var/lib/kubelet/config.yaml` look for `staticPodPath`

  or

- `--pod-manifest-path=/etc/kubernetes/manifests/` **(command line method for creating the static pod)**
  - `systemctl restart kubelet`
- `kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml`

```shell
controlplane ~ ➜  ls -lrt /etc/kubernetes/manifests/
total 16
-rw------- 1 root root 2406 Apr  7 16:35 etcd.yaml
-rw------- 1 root root 1463 Apr  7 16:35 kube-scheduler.yaml
-rw------- 1 root root 3393 Apr  7 16:35 kube-controller-manager.yaml
-rw------- 1 root root 3882 Apr  7 16:35 kube-apiserver.yaml
```

- to ssh into any node within k8s cluster you can use the command `ssh <Node-name>/<Node-IP>`
