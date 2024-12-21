# [Taints and Tolerence](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

- Taints are placed on Nodes
- Tolerations is placed on Pods
- Tolerations allow the scheduler to schedule pods with matching taints.
- if Taint are placed on the specific node it will not allow a pod without tolerence tobe scheduled on that Node
- if we place the tolerence on the specific pod then only it will be able to scheduled on the node else not
- Taints are the opposite -- they allow a node to repel a set of pods.
- Taints and tolerations work together to ensure that pods are not scheduled onto **inappropriate** nodes.
- One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints
- Taints and Tolerations does not tell pod to go on specific node
- _Taints and Tolerations only tells accept pods on the node with certain tolerations._
- if you want to restrict a certain pod on a single node then you have to use the node **Affinity**.
- Taint is alredy placed on masterNode which tells the scduler not to schedule pods on the masterNode to view which taint effect is applied on the master node use below command.
  - `kubectl describe node kubemaster | grep Taint`

## Taint-Effects

- **NoExecute**
  - Pods that do not tolerate the taint are evicted immediately
  - Pods that tolerate the taint without specifying `tolerationSeconds` in their toleration specification remain bound forever
  - Pods that tolerate the taint with a specified `tolerationSeconds` remain bound for the specified amount of time. After that time elapses, the node lifecycle controller evicts the Pods from the node
  - Pods currently running on the node are **evicted** if does not match the tolerence.
  - `NoExecute` effect can specify an optional `tolerationSeconds` field that dictates how long the pod will stay bound to the node after the taint is added
- **NoSchedule**
  - No new Pods will be scheduled on the tainted node unless they have a matching toleration. Pods currently running on the node are not evicted.
- **PreferNoSchedule**
  - `PreferNoSchedule` is a "preference" or "soft" version of `NoSchedule`.
  - The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, _but it is not guaranteed_.

---

### Commands: (Applying Tolerence to Node:)

To Apply the Taint on the Node

`kubectl taint nodes node1 key1=value1:NoSchedule`

To remove the taint from the node:

`kubectl taint nodes node1 key1=value1:NoSchedule-`

```
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

### Applying Tolerence to Pod

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
      env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent

# add tolerence to Pod
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```

---