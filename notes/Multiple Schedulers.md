# [Multiple Schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

- K8s comes with a single default scheduler but as the k8s is highly extensible, If the default scheduler does not suit your needs you can **implement your own scheduler**.
- you can even run multiple schedulers simultaneously alongside the default scheduler.

---

**STEP 1:**

- Package your scheduler binary into a container image.
- Clone the [Kubernetes](https://github.com/kubernetes/kubernetes.git) source code from GitHub and build the source.
- `Docker Build -t` and `Docker Push` to push the Image to Docker Hub (or another registry). refer k8s documentation

---

**Docker File**

```Dockerfile
FROM busybox
ADD ./_output/local/bin/linux/amd64/kube-scheduler /usr/local/bin/kube-scheduler
```

---

**STEP 2:** Define a Kubernetes Deployment for the scheduler

- you have your scheduler in a container image
- create a pod/Deployment configuration for it and run it in your Kubernetes cluster.
- Save it as [my-scheduler.yaml](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/sched/my-scheduler.yaml)
- in the deployment use this

```yml
---
# **my-scheduler-config.yaml**
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
leaderElection:
  leaderElect: false

---
# **my-scheduler-configmap.yaml**
apiVersion: v1
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-scheduler-config
  namespace: kube-system

---
# **my-scheduler.yml**
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  serviceAccountName: my-scheduler
  containers:
  - command:
    - /usr/local/bin/kube-scheduler
    - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
    image: registry.k8s.io/kube-scheduler:v1.29.0
    livenessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
    name: kube-second-scheduler
    readinessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
    resources:
      requests:
        cpu: '0.1'
    securityContext:
      privileged: false
    volumeMounts:
      - name: config-volume
        mountPath: /etc/kubernetes/my-scheduler
  hostNetwork: false
  hostPID: false
  volumes:
    - name: config-volume
      configMap:
        name: my-scheduler-config
```

---

**STEP 3:** How to use the custom schduler into pod definations.

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx

  # This is how you can add the custom scheduler to pod.
  schedulerName: my-scheduler
```

**Example**:

```
controlplane ~ ➜  kubectl get pods -A
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-nssdp                  1/1     Running   0          10m
kube-system    coredns-69f9c977-25wxh                 1/1     Running   0          10m
kube-system    coredns-69f9c977-9v84m                 1/1     Running   0          10m
kube-system    etcd-controlplane                      1/1     Running   0          11m
kube-system    kube-apiserver-controlplane            1/1     Running   0          11m
kube-system    kube-controller-manager-controlplane   1/1     Running   0          10m
kube-system    kube-proxy-zhj22                       1/1     Running   0          10m
kube-system    kube-scheduler-controlplane            1/1     Running   0          10m
kube-system    **my-scheduler**                           1/1     Running   0          2m22s
```

---

`kubectl get events -o wide`

`kubectl logs <scheduler_name> -n <name_space>`

---
# [Scheduler Profiles](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)

- Suppose if you have multiple schedulers you can combine all those schedulers to use same binary rather than creating multiple binaries, this can be achived with the help of Scheduler Profile in latest k8s versions.
- this is a efficient way to use and configure mutiple schedulers in k8s cluster.
- How a Pod gets scheduled on the nodes
  - **Sceduling Queue**
  - **Filtering**
  - **Scoring**
  - **Binding**

![alt text](<Screenshot 2024-04-20 at 13-52-52 Certified Kubernetes Administrator (CKA) Practice Exam Tests.png>)

## Scheduling Queue

### PriorityClass

- you can set the priority of pod tobe scheduled

```yml
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."

---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
  resources:
    requests:
      memory: 1Gi
      cpu: 10
```

## [Filtering](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/#kube-scheduler-implementation)

- Filter which nodes are not suitable for running the pod, e.g., lack of resource or taints and tolerations

## [Scoring](https://discuss.kubernetes.io/t/kube-pod-scoring-criteria/20743)

- after the filtering phase Nodes which are eligible will go through the scoring phase . The node with the highest score is selected.
- Score means Suitable to run the Pod and have more available free resource after scheduling.
- The scheduler assigns a score to each Node that survived filtering, basing this score on the active scoring rules.

## Binding

- The nodes which have a good score are selected for binding.

### [Sceduler Plugins](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/#interfaces)

- **Scheduling Queue**
  - PrioritySort
- **Filtering**
  - NodeResourcesFit
  - NodeName
  - NodeUnschedule
- **Scoring**
  - NodeResourcesFit
  - ImageLocality
- **Binding**
  - DefaultBinder

These Plugins are pluged to **Extention points**

1. queueSort
2. filter
3. score
4. bind

---
