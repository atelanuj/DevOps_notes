# PersistentVolumeClaim (PVC)

- A PersistentVolumeClaim is a request for storage by a user. It specifies size, access modes, and optionally, a StorageClass. Kubernetes matches PVCs to PVs based on these specifications

**Key Attributes**:

1. **Requested Storage**: The size of the storage needed.
2. **Access Modes**: How the storage should be accessed.
3. **StorageClass**: (Optional) Specifies the desired StorageClass for dynamic provisioning.
4. every `PVC` binds to a single `PV` k8s finds `PV` with sufficient requirements. you can also use the `labels` to bind to the right volume.
5. if no new `PV` is available then `PVC` will go in `pending` state.
6. multiple `PVCs` cannot bind to the same `PV`.
   1. A PVC (Persistent Volume Claim) is exclusively bound to a PV (Persistent Volume). The binding is a one-to-one mapping, using a ClaimRef, which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-sc # Optinal
```

`persistentVolumeReclaimPolicy` has three option `retain`, `recycle` and `deleted`

The `persistentVolumeReclaimPolicy` in Kubernetes specifies what should happen to a `PersistentVolume (PV)` when its associated PersistentVolumeClaim (PVC) is deleted.

- **`Retain`**
  **Use Case:** Useful when data should be preserved after a Pod stops using it, such as for auditing or backup purposes. Administrators can later manually manage the volume (e.g., delete or reassign it to a new claim).
- **`Delete`**
  **Use Case:** Common in cloud environments where dynamically provisioned volumes are frequently created and removed. Ideal when storage needs to be automatically freed up to avoid unnecessary costs.
- **`Recycle`** _(**Deprecated** in many Kubernetes versions)_
  **Use Case:** Used in older Kubernetes setups where basic cleansing of data on release was acceptable. It is now mostly replaced by more secure methods.

_how to use PVC inside the POD defination_

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx

    # This subsection defines how volumes are mounted into the container.
    volumeMounts:

    # mount name
    - name: my-storage

      # Mount Path inside container
      mountPath: "/usr/share/nginx/html"
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-manual-pvc
```

> Note: `spec.containers.volumeMounts.name` must be same as `spec.volumes.name` and policy should be same for both `PV` and `PVC` to bind else the pvc will be in `pending` state, if pod is using the `PVC` and you delete the `PVC` then the `PVC` will stuck in **terminating** state once the `pod` is deleted `pvc` will also delete and `PV` will be **released**

# [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)

- A StorageClass provides a way to describe the "classes" of storage available in a Kubernetes cluster.
- **StorageClasses help Kubernetes _dynamically_ provision PVs.**
- If you are leveraging dynamic provisioning, you should create the StorageClass first. This is because the StorageClass is responsible for dynamically provisioning the PersistentVolumes when a PersistentVolumeClaim (PVC) is created.
- A StorageClass provides a way for administrators to describe the classes of storage they offer
- It abstracts the underlying storage provider (like AWS EBS, GCE PD, NFS, etc.) and allows dynamic provisioning of PVs.
- kubernetes communicates with the **Cloud providers API** to provisions a volume.

**Key Attributes**:

1. **Provisioner**: Specifies the volume plugin (e.g., kubernetes.io/aws-ebs, kubernetes.io/gce-pd, kubernetes.io/nfs).
2. **Parameters**: Configuration options specific to the provisioner.
3. **Reclaim Policy**: Defaults for dynamically provisioned PVs.

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
parameters: # This is provider specific
  type: gp2
  zones: us-east-1a,us-east-1b
reclaimPolicy: Retain
```

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-sc
```

**Workflow**:

- **Create the StorageClass**: Define the storage characteristics and provisioner.
- **Create the PVC**: Reference the StorageClass in the PVC specification.
- **Kubernetes Automatically Creates the PV**: Kubernetes uses the provisioner to create the storge.

### Static vs Dynamic provisioning of PV in K8s

#### **Dynamic Provisioning**

- **StorageClass First**: If you're using dynamic provisioning, you should create the StorageClass first. This is because when a PersistentVolumeClaim (PVC) is created with a storageClassName, Kubernetes uses the StorageClass to dynamically provision a PersistentVolume that matches the PVC's requirements.
- **PersistentVolume (PV)** : This will be automatically created by `StorageClass`
- **PVC Creation Triggers PV Creation**: After the StorageClass is defined, the PVC can be created. Kubernetes will then automatically create a PV based on the specifications in the StorageClass and bind it to the PVC.
  - Give `StorageClass` name in `PVC` defination

**Order**:

- StorageClass
- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)

#### **Static Provisioning**

- **PersistentVolume First**: In the case of static provisioning, you manually create the PVs first. These PVs are pre-provisioned by an administrator and are available in the cluster for any PVC to claim.
- **PVC Creation Binds to PV**: After the PVs are created, you can create PVCs that match the specifications of the available PVs. Kubernetes will bind the PVC to a matching PV.

**Order**:

- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)

---
## Imperative Commands to find pods with selected Labels

`kubectl get pods --selector key=value --no-headers | wc -l`

`kubectl get pods --selector env=prod,env=prod,tier=frontend`