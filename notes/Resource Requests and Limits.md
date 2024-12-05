# [Resource Requests and Limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

- **Requests**: Requests specify the minimum amount of resources (CPU and memory) that a pod needs to run.

  - When a pod is scheduled onto a node, Kubernetes ensures that the node has enough available resources to satisfy the pod's resource requests.
  - Requests are used by the **scheduler** to determine the best node for placing a pod.

- **Limits**: Limits, on the other hand, specify the maximum amount of resources that a pod can consume.
  - Kubernetes enforces these limits to prevent pods from using more resources than allowed, which helps in maintaining stability and preventing resource contention.
  - If a pod exceeds its specified limits, Kubernetes may take actions such as throttling or terminating the pod.
  - Pod cannot exceed the CPU limit but can exceed the Memory Limit in that case K8s will try to Terminate or kill the Pod when it finds that the pod is using more resources than allowed `OOM` (out of memory (OOM) error) `OOMKilled` means the pod has been killed by Out of Memory.
  - if you donot specify the Request the Limit will be `Request = Limit`

## Use Cases

1. **No Request and No Limit**

- This scenario is suitable for non-critical applications or experiments where resource requirements are flexible, and the pod can utilize whatever resources are available on the node without any constraints. However, it can lead to potential resource contention issues if other pods on the node require resources.

2. **No Request and Limit**

- This scenario is useful when you want to constrain the resource usage of a pod to prevent it from using excessive resources and impacting other pods on the node. However, without a request, Kubernetes scheduler may place the pod on a node without considering its resource needs, potentially leading to inefficient resource allocation.

3. **Request and Limit**

- This is the most common and recommended configuration. By specifying both requests and limits, Kubernetes can ensure that the pod gets scheduled on a node with sufficient resources and enforce resource constraints to maintain stability and fairness within the cluster.

4. **Request and No Limits**

- This scenario is suitable when you want to ensure that the pod gets scheduled on a node with sufficient resources based on its requirements, but you're not concerned about limiting its resource usage. It allows the pod to scale up its resource consumption based on demand without strict constraints, which can be useful for certain workloads with variable resource requirements.

---

```yml
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

> Kubernetes doesn't allow you to specify CPU resources with a precision finer than `1m or 0.001 CPU`. To avoid accidentally using an invalid CPU quantity, it's useful to specify CPU units using the milliCPU form instead of the decimal form when using less than `1 CPU` unit.
> For example, you have a Pod that uses `5m or 0.005 CPU` and would like to decrease its CPU resources. By using the decimal form, it's harder to spot that `0.0005 CPU` is an invalid value, while by using the milliCPU form, it's easier to spot that `0.5m` is an invalid value.
> Limits and requests for `memory` are measured in bytes. You can express memory as a plain integer or as a fixed-point number using one of these quantity suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent roughly the same value:
> `128974848, 129e6, 129M,  128974848000m, 123Mi`
> Pay attention to the case of the suffixes. If you request `400m` of memory, this is a request for 0.4 bytes. Someone who types that probably meant to ask for 400 mebibytes (`400Mi`) or 400 megabytes (`400M`).

---

# [DeamonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

- DeamonSet are simillar to ReplicaSets but inly difference is that DeamonSet ensure that every node has at least one instance of a pod running.
- when the node gets created DeamonSet automatically creates the pod on the Node.
- Pods created by the DeamonSet are ignored by the Kube-scheduler.
- deamonSets does not support scalling inside the Node
- it supports the Rolling update like Deployments.

## Use Cases

- running a cluster storage daemon on every node
- running a _logs collection_ daemon on every node
- running a _node monitoring_ daemon on every node
- one DaemonSet, covering all nodes, would be used for each type of daemon.
  ![alt text](image.png)

```yml
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

> If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that **node selector**. Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that **node affinity**. If you do not specify either, then the DaemonSet controller will create Pods on all nodes.

---
