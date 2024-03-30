# [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/):
- Labels are the way by which you can filter and sort the number pods in a cluster
- selector tells the replicaset/controller/deployment which pods to watch/belongs to
- The label selector is the core grouping primitive in Kubernetes.
- Labels are usedful for filtering the environment specific pods 
- You can assign multiple labels to a pod, node, cluster objects.
- By giving multiple labels you can filter out exact pods that you required.
- Labels are key=value pair without any predefined meaning 
- They are similar to tags in AWS and Git
- You are free to choose labels of any name.
- You can provide label through both imperative and declarative methods.
- Define it under the **metadata**
- **Annotations** are extra details for the pod
---
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: myDeployment

    # This are the Deployment labels 
    labels:
        app: myApp
        env: dev
spec:
   replicas: 3

   # the Selector selects the pod with specific labels
   selector: 
      matchLabels:
      tier: front-end
   
   # template  defines the PodSpec for the new pods being created and what Labels it contains
   template:
      metadata: 
      name: testpod
      labels:
          tier: front-end
      spec:
      containers:
      - name: c00
          image: nginx:latest
```
---

## Imperative Commands to find pods with selected Labels
`kubectl get pods --selector key=value --no-headers | wc -l`

`kubectl get pods --selector env=prod,env=prod,tier=frontend`

# [Taints and Tolerence](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/):
- Taints are placed on Nodes
- Tolerations is placed on Pods
- Tolerations allow the scheduler to schedule pods with matching taints.
- if Taint are placed on the specific node it will not allow a pod without tolerence tobe scheduled on that Node
- if we place the tolerence on the specific pod then only it will be able to scheduled on the node else not 
- Taints are the opposite -- they allow a node to repel a set of pods.
- Taints and tolerations work together to ensure that pods are not scheduled onto **inappropriate** nodes.
- One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints
- Taints and Tolerations does not tell pod to go on specific node
- *Taints and Tolerations only tells accept pods on the node with certain tolerations.* 
- if you want to restrict a certain pod on a single node then you have to use the node **Affinity**. 
- Taint is alredy placed on masterNode which tells the scduler not to schedule pods on the masterNode to view which taint effect is applied on the master node use below command.
  - `kubectl describe node kubemaster | grep Taint`

## Taint-Effects:
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
  - The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, *but it is not guaranteed*.
---
### Commands: (Applying Tolerence to Node:)
To Apply the Taint on the Node

`kubectl taint nodes node1 key1=value1:NoSchedule`

To  remove the taint from the node:

`kubectl taint nodes node1 key1=value1:NoSchedule-`
```
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

### Applying Tolerence to Pod:
```
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
# [Node Selectors](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/):
- Node selectors helps in scheduling the pod on specific node based on labels assigned to node and the pods
- suppose we have Three nodes namely Large, Small with labels as size = large/small.
- We can use this label information to schedule our pods accordingly.

Example :
```
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

# Apply node selector to run pod on node with size=large label on it
  nodeSelector:
    size: large
```
## Commands:
Add a label to a particular node

`kubectl label nodes <node_name> key=value`

### Limitations:
> NodeSelector will not be able to help in complex senerios such as
> - size=large or size=small
> - size !=medium (not equal)

---
# [Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/):
- with the help of Node Affinity , you can control the scheduling of pods  onto nodes more granularly using various criteria like
- It is an extension of node selectors which allows you to specify more flexible rules about where to place your pods by specifying certain requirements.

Example: This manifest describes a Pod that has a `requiredDuringSchedulingIgnoredDuringExecution` node affinity,`size=large or size=small`. This means that the pod will get scheduled only on a node that has a `size=large or size=small`
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values: # The Pod will scheduled on the nodes that have at least one of these value.
            - large
            - small            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```
## Node Affinity types:
**`required`DuringScheduling`Ignored`DuringExecution**

| Syntax | DuringScheduling | DuringExecution |
| ----------- | ----------- | --------------- |
| Type1 | Required | Ignored | 
| Type2 | preferred | Ignored |
| Type3 | Required | Required |