# [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

- StatefulSet is a workload API object used to manage stateful applications.
- A StatefulSet runs a group of Pods, and maintains a sticky identity for each of those Pods. This is useful for managing applications that need persistent storage or a stable, **unique network identity**
- Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods
- StatefulSet maintains a sticky identity for each of its Pods.
- Ensures Pods are created, updated, or deleted in a specific order.
- Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.
- **StatefulSets are valuable for applications that require one or more of the following.**
  - Stable, unique network identifiers.
  - Stable, persistent storage.
  - Ordered, graceful deployment and scaling.
  - Ordered, automated rolling updates
- The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner ([examples here](https://github.com/kubernetes/examples/blob/master/staging/persistent-volume-provisioning/README.md)) based on the requested storage class, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet.
- StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.

```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi

```
## Volume Claim Templates

- You can set the `.spec.volumeClaimTemplates` field to create a `PersistentVolumeClaim`.
- StatefulSet Pods have a **unique identity** that consists of an ordinal, a stable network identity, and stable storage. The identity sticks to the Pod, regardless of which node it's (re)scheduled on.

## StatefulSets Diagrams

![alt text](statefulset_pvc.png)
![alt text](image-19.png)

## StatefulSets vs Deployment

Kubernetes offers multiple ways to manage application workloads, including **StatefulSets** and **Deployments**. Each is designed for specific use cases and provides different functionalities. Below is a detailed comparison.

### 1. Overview

- **StatefulSet**: Manages stateful applications. Ensures that each Pod in a set has a unique, persistent identity and stable storage.
- **Deployment**: Primarily for stateless applications. Manages a set of Pods where each Pod is generally identical to any other Pod in the set.

### 2. Use Cases

- **StatefulSet**:
  - Suitable for applications requiring stable, unique network identities.
  - Used when persistent storage is required across Pod restarts.
  - Ideal for databases (e.g., MongoDB, Cassandra), distributed systems (e.g., Kafka), and applications where Pods need ordered startup/shutdown.
- **Deployment**:
  - Used for stateless applications where any Pod can handle any request.
  - Suitable for replicated services (e.g., web servers, REST APIs).
  - Typically used when Pods do not require stable network identities or storage persistence.

### 3. Pod Identity and Ordering

- **StatefulSet**:
  - Each Pod has a unique, stable network identity and hostname.
  - Pods are named sequentially (e.g., `podname-0`, `podname-1`, etc.).
  - Provides **ordered, deterministic** deployment and scaling.
- **Deployment**:
  - Pods are interchangeable, with no unique identities.
  - No guaranteed ordering during scaling up or down.
  - Pods are deployed and removed in a random order.

### 4. Storage and Persistence

- **StatefulSet**:
  - Uses PersistentVolumeClaims (PVCs) to ensure each Pod has a stable, unique storage that persists across rescheduling.
  - Each Pod can have its own PVC, ensuring persistent storage tied to the Pod’s identity.
- **Deployment**:
  - Typically does not provide persistent storage for Pods (unless used with shared PersistentVolumes).
  - Pods do not maintain unique, stable storage; volumes are usually shared among Pods if necessary.

### 5. Scaling Behavior

- **StatefulSet**:
  - Scaling is done in an ordered fashion (Pod `n+1` is only created after Pod `n` is ready).
  - Scaling down happens in reverse order, which is useful for maintaining application consistency.
- **Deployment**:
  - Pods are added or removed randomly without ordering constraints.
  - Suitable for scaling stateless applications without the need for ordered shutdown/startup.

### 6. Rolling Updates

- **StatefulSet**:
  - Supports rolling updates but with stricter controls.
  - Allows gradual updates with controlled ordering to ensure stable state across updates.
- **Deployment**:
  - Supports rolling updates by default with quicker, more flexible scaling and rollout.
  - Ideal for fast updates where application state or order is not critical.

### 7. Failover and Recovery

- **StatefulSet**:
  - Pods have stable identities, so recovery is deterministic and consistent.
  - Pods retain their identity on failover, maintaining state consistency.
- **Deployment**:
  - Any Pod can replace another upon failure, which may lead to quicker recovery for stateless apps.
  - Statelessness makes it easier to handle failure without risking data consistency.

### 8. Examples

- **StatefulSet**: Databases (MySQL, MongoDB), messaging queues (Kafka, RabbitMQ), distributed file systems (HDFS).
- **Deployment**: Web applications, API servers, microservices, load-balanced services.

### 9. Configuration Snippet

#### StatefulSet Example

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  selector:
    matchLabels:
      app: myapp
  serviceName: "myapp-service"
  replicas: 3
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp-image
        volumeMounts:
        - name: myapp-storage
          mountPath: "/data"
  volumeClaimTemplates:
  - metadata:
      name: myapp-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

#### Deployment Example

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp-image
        ports:
        - containerPort: 80
```

# [PersistentVolume (PV) Provisioning](https://github.com/kubernetes/examples/blob/master/staging/persistent-volume-provisioning/README.md)

- A PersistentVolume is a piece of storage in the Kubernetes cluster that has been _provisioned by an administrator_ or _dynamically using StorageClasses_.
- The admin must define StorageClass objects that describe named "classes" of storage offered in a cluster.
- When configuring a StorageClass object for persistent volume provisioning, the admin will need to describe the type of **provisioner** to use and the **parameters** that will be used by the provisioner when it provisions a PersistentVolume belonging to the class.
- The provisioner field must be specified as it determines what `volume plugin` is used for provisioning PVs.
- The parameters field contains the parameters that describe volumes belonging to the storage class. Different parameters may be accepted _depending on the provisioner_.

**Key Attributes:**

- **Capacity**: The size of the storage.
- **Access Modes**: Defines how the volume can be mounted (e.g., ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
- **Reclaim Policy**: Determines what happens to the PV after the PVC is deleted (e.g., Retain, Recycle, Delete).
- **StorageClass**: Links the PV to a StorageClass for dynamic provisioning.

AWS EBS PV

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  csi.storage.k8s.io/fstype: xfs
  type: io1
  iopsPerGB: "50"
  encrypted: "true"
allowedTopologies:
- matchLabelExpressions:
  - key: topology.ebs.csi.aws.com/zone
    values:
    - us-east-2c
```

```yml
---
apiVersion: v1
kind: PersistantVolume
metadata:
  name: example-volume
  labels:
    type: local
spec:
  storageClassName: ebs-sc
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  hostpath:
    path: "/mnt/data"
```

**\*\*\*\***PersistantVolume without StorageClass.**\*\*\*\***

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-manual-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain

  # Adding a NFS PV
  nfs:
    server: nfs-server.default.svc.cluster.local
    path: "/path/to/data"

  # Or you can add existing EBS volume as PV
  awsElasticBlockStore:
    volumeID: <volume-id>
    fstype: ext4
```

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```