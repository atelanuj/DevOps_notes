# [Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)

- with the help of Node Affinity , you can control the scheduling of pods onto nodes more granularly using various criteria like
- It is an extension of node selectors which allows you to specify more flexible rules about where to place your pods by specifying certain requirements.

Example: This manifest describes a Pod that has a `requiredDuringSchedulingIgnoredDuringExecution` node affinity,`size=large or size=small`. This means that the pod will get scheduled only on a node that has a `size=large or size=small`

```yml
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

## Node Affinity types

1. **`required`DuringScheduling`Ignored`DuringExecution**
   - Pods with this affinity rule will only be scheduled on nodes that meet the specified requirements. However, if a node loses the label that made it eligible after the pod is scheduled, the pod will still continue to run on that node.
2. **`Preferred`DuringScheduling`Ignored`DuringExecution**
   - Pods with this affinity rule will prefer to be scheduled on nodes that meet the specified requirements, but they are not strictly required to. If no nodes match the preferred requirements, the pod can still be scheduled on other nodes.

| Syntax | DuringScheduling | DuringExecution |
| ------ | ---------------- | --------------- |
| Type1  | Required         | Ignored         |
| Type2  | preferred        | Ignored         |
| Type3  | Required         | Required        |

### Taints and Tolerence & Node Affinity

- You can use both taints and tolerations & Node affinity to granular control over the Pod scheduling on the specific nodes.

- **what if we applied taint to specific node and Node Affinity to a pod to be schedule on the specific node only without tolerance set what will happen?**
  - When the scheduler attempts to place the pod onto a node, it evaluates the Node Affinity rules against the labels of each node in the cluster.
  - If there is a node that meets the Node Affinity requirements specified in the pod's configuration and doesn't have a taint that repels the pod (i.e., the node matches the Node Affinity and doesn't have any taints), the pod will be scheduled on that node.
  - However, if the node with the matching Node Affinity also has the taint that the pod lacks tolerations for, the pod won't be scheduled on that node due to the taint-repelling behavior. In this case, the pod won't be scheduled anywhere unless there is another node in the cluster that meets the Node Affinity requirements and doesn't have the taint applied.

---
