# Multicontainer pods

- The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster.
- The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.

## Multi Container Pods design and Pattern

1. SIDECAR
2. ADAPTER
3. AMBASSADOR

![alt text](image-7.png)

---

#### SideCar Container

#### ADAPTER Container

#### AMBASSADOR Container

## Pods in a Kubernetes cluster are used in two main ways

- Pods that run a single container.
  - The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.
- Pods that run multiple containers that need to work together.
  - A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit of service—for example, one container serving data stored in a shared volume to the public, while a separate **sidecar** container refreshes or updates those files. The Pod wraps these containers, storage resources, and an ephemeral network identity together as a single unit.

### [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

- Init containers can contain **utilities or setup scripts** not present in an app image.
- You can specify init containers in the Pod specification alongside the containers array (which describes app containers).
- containers that run to completion during Pod initialization.
- init containers, which are run before the app containers are started.
- If a Pod's init container `fails`, the `kubelet` repeatedly restarts that init container until it succeeds.
- However, if the Pod has a `restartPolicy` of `Never`, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.
- But at times you may want to run a process that runs to completion in a container.

**For example** a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only one time when the pod is first created. Or a process that waits for an external service or database to be up before the actual application starts. That's where initContainers comes in.

Example 1:

```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

**Init containers are exactly like regular containers, except:**

- Init containers always run to completion.
- Each init container must `complete successfully` before the next one starts.

### [Differences from regular containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#differences-from-regular-containers)

- Init containers support all the fields and features of app containers, including resource limits, volumes, and security settings
- the resource requests and limits for an init container are handled differently
- Regular init containers (in other words: excluding sidecar containers) do not support the lifecycle, livenessProbe, readinessProbe, or startupProbe fields.

### Differences in `init` and `Sidecar` cinatiner

**Init container**

- They run to completion before any application containers start within a pod.
- Their primary role is to set up the correct environment for the app to run.
- This may involve tasks like database migrations, configuration file creation, or permission setting.
- Once an init container has successfully completed its task, the app containers in the pod can start.
- init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe
- Init containers share the same resources (CPU, memory, network) with the main application containers

  **Sidecar**

- They run alongside the main application container, providing additional capabilities like logging, monitoring, or data synchronization.
- Unlike init containers, sidecar containers remain running for the life of the pod.
- They are used to enhance or extend the functionality of the primary app container without altering the primary application code.
- For example, a logging sidecar might collect logs from the main container and forward them to a central log store.
- whereas sidecar containers support all these probes to control their lifecycle.

### Mutiple `Init` conatainers

- If you specify multiple init containers for a Pod, kubelet runs each init container sequentially
- Each init container must succeed before the next can run.
- When all of the init containers have run to completion, `kubelet` initializes the application containers for the Pod and runs them as usual.
- If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
- if the pod fails to restart after sevral attemps then K8s gives up and pod goes into `crashloopbackoff` error.

Example 2:

```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

```
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
```
