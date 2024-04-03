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
1. **`required`DuringScheduling`Ignored`DuringExecution**
     - Pods with this affinity rule will only be scheduled on nodes that meet the specified requirements. However, if a node loses the label that made it eligible after the pod is scheduled, the pod will still continue to run on that node.
2. **`Preferred`DuringScheduling`Ignored`DuringExecution**
     -  Pods with this affinity rule will prefer to be scheduled on nodes that meet the specified requirements, but they are not strictly required to. If no nodes match the preferred requirements, the pod can still be scheduled on other nodes.
  

| Syntax | DuringScheduling | DuringExecution |
| ----------- | ----------- | --------------- |
| Type1 | Required | Ignored | 
| Type2 | preferred | Ignored |
| Type3 | Required | Required |

### Taints and Tolerence & Node Affinity:
- You can use both  taints and tolerations & Node affinity to granular control over the Pod scheduling on the specific nodes.

- **what if we applied taint to specific node and Node Affinity to a pod to be schedule on the specific node only without tolerance set what will happen?**
  - When the scheduler attempts to place the pod onto a node, it evaluates the Node Affinity rules against the labels of each node in the cluster.
  - If there is a node that meets the Node Affinity requirements specified in the pod's configuration and doesn't have a taint that repels the pod (i.e., the node matches the Node Affinity and doesn't have any taints), the pod will be scheduled on that node.
  - However, if the node with the matching Node Affinity also has the taint that the pod lacks tolerations for, the pod won't be scheduled on that node due to the taint-repelling behavior. In this case, the pod won't be scheduled anywhere unless there is another node in the cluster that meets the Node Affinity requirements and doesn't have the taint applied.

---
# [Resource Requests and Limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/):
- **Requests**: Requests specify the minimum amount of resources (CPU and memory) that a pod needs to run.
  - When a pod is scheduled onto a node, Kubernetes ensures that the node has enough available resources to satisfy the pod's resource requests.
  - Requests are used by the **scheduler** to determine the best node for placing a pod.

- **Limits**: Limits, on the other hand, specify the maximum amount of resources that a pod can consume.
  - Kubernetes enforces these limits to prevent pods from using more resources than allowed, which helps in maintaining stability and preventing resource contention.
  - If a pod exceeds its specified limits, Kubernetes may take actions such as throttling or terminating the pod.
  - Pod cannot exceed the CPU limit but can exceed the Memory Limit in that case K8s will try to Terminate or kill the Pod when it finds that the pod is using more resources than allowed `OOM` (out of memory (OOM) error) `OOMKilled` means the pod has been killed by Out of Memory.
  - if you donot specify the Request the Limit will be `Request = Limit`

## Use Cases:
1. **No Request and No Limit**
  - This scenario is suitable for non-critical applications or experiments where resource requirements are flexible, and the pod can utilize whatever resources are available on the node without any constraints. However, it can lead to potential resource contention issues if other pods on the node require resources.

2. **No Request and Limit**
  - This scenario is useful when you want to constrain the resource usage of a pod to prevent it from using excessive resources and impacting other pods on the node. However, without a request, Kubernetes scheduler may place the pod on a node without considering its resource needs, potentially leading to inefficient resource allocation.

3. **Request and Limit**
  - This is the most common and recommended configuration. By specifying both requests and limits, Kubernetes can ensure that the pod gets scheduled on a node with sufficient resources and enforce resource constraints to maintain stability and fairness within the cluster.

4. **Request and No Limits**
  - This scenario is suitable when you want to ensure that the pod gets scheduled on a node with sufficient resources based on its requirements, but you're not concerned about limiting its resource usage. It allows the pod to scale up its resource consumption based on demand without strict constraints, which can be useful for certain workloads with variable resource requirements.
---
```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "1"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
**Note**:
>Kubernetes doesn't allow you to specify CPU resources with a precision finer than `1m or 0.001 CPU`. To avoid accidentally using an invalid CPU quantity, it's useful to specify CPU units using the milliCPU form instead of the decimal form when using less than `1 CPU` unit.
>For example, you have a Pod that uses `5m or 0.005 CPU` and would like to decrease its CPU resources. By using the decimal form, it's harder to spot that `0.0005 CPU` is an invalid value, while by using the milliCPU form, it's easier to spot that `0.5m` is an invalid value.
>Limits and requests for `memory` are measured in bytes. You can express memory as a plain integer or as a fixed-point number using one of these quantity suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent roughly the same value:
  `128974848, 129e6, 129M,  128974848000m, 123Mi`
>Pay attention to the case of the suffixes. If you request `400m` of memory, this is a request for 0.4 bytes. Someone who types that probably meant to ask for 400 mebibytes (`400Mi`) or 400 megabytes (`400M`).
---
# [DeamonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/):
- DeamonSet are simillar to ReplicaSets but inly difference is that DeamonSet ensure that every node has at  least one instance of a pod running.
- when the node gets created DeamonSet automatically creates the pod on the Node.
    ## Use Cases:
    - running a cluster storage daemon on every node
    - running a logs collection daemon on every node
    - running a node monitoring daemon on every node
- one DaemonSet, covering all nodes, would be used for each type of daemon.
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```
`kubectl apply -f DeamonSet.yml`

## Running Pods on select Nodes 
>If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that **node selector**. Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that **node affinity**. If you do not specify either, then the DaemonSet controller will create Pods on all nodes.