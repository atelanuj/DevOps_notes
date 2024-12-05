## [Authorizations](https://kubernetes.io/docs/reference/access-authn-authz/authorization/):

- Kubernetes authorization takes place following authentication. Usually, a client making a request must be authenticated (logged in) before its request can be allowed; however, Kubernetes also allows anonymous requests in some circumstances.
- All parts of an API request must be allowed by some `authorization mechanism` in order to proceed. In other words: access is denied by default.

## [Authorization modes](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules)

1. RBAC
2. ABAC
3. Webhook
4. Node

## ![alt text](image-18.png)

### RBAC (Role based access control)

- RBAC is the default authorization mode in Kubernetes.
- It is based on the concept of roles and bindings.
- Roles are collections of permissions.
- Bindings are used to assign roles to users or groups.
- in RBAC permissions are not assign directly to the user or group.
- `Role` and `RoleBinding` are namespace specific.
- A Role always sets permissions within a particular namespace; when you create a Role, you have to specify the namespace it belongs in.
- for cluster level you need to use the `ClusterRole`and `ClusterRoleBinding` which covers all namespaces.
- to check which Authorization modes are enabled on the cluster you can use the
  - `kubectl describe pod kube-apiserver-controlplane -n kube-system | grep "--authorization-mode"`

---

1. Define a `generic container` for permissions : a `Role`.
2. Assign the `permissions` to the `Role`.
3. Bind the `Role` to the `User` and `Group`.

---

- ![alt text](image-17.png)

---

### Role and RoleBinding

**Example:**

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

To link the User to the `Role` we need to use the `RoleBinding`.

```yml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

- RoleBinding also operates on the `namespace` level.

**commands**

- `kubectl get roles` to list all roles in the current namespace
- `kubectl get rolebindings`
- `kubectl describe role <role_name>`

**Check acess**

To see if the use has the access to a perticuler resource in the cluster

- kubectl auth `can-i` create deployment ----> yes
- kubectl auth `can-i` delete nodes ------> no
- kubectl auth `can-i` create deployment --as `dev-user` ----> no
- kubectl auth `can-i` create pods --as `dev-user` -----> yes
- kubectl auth `can-i` create pods --as `dev-user` --namespace test ----> no

---

### ClusterRole and ClusterRoleBindings

- `ClusterRole` and `ClusterRoleBindings` are used to manage access at the cluster level. They are similar to `Role` and `RoleBindings`.
- with the help of CLusterRole the `user` can access the `resources` of other namespaces too.
- _Use-cases_
  - **Node**
    - delete Node
    - Create Node
    - update Node
  - **Storage**
    - delete PV
    - Create PV
    - update PV

**Namespaced Resources**

- Role
- RoleBindings
- Deployments
- Pods
- ReplicaSets
- ConfigMaps
- Secretes
- Services
- PVC
- Jobs

**Cluster Scoped Resources**

- ClusterRole
- ClusterRoleBindings
- PV
- nodes
- CertificateSingningRequests
- namespaces

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

```yml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### [ServiceAccounts](https://kubernetes.io/docs/concepts/security/service-accounts/)

- ServiceAccount is used when an application want to connect or retrieve data from the cluster
  - Authenticating to the API server or implementing identity-based security policies.
- **There are two types of accounts in k8s:**
  - **User Account**
    - used by users
    - admin access the cluster to administer the cluster
    - developer access the cluster to deploy the application
    - By default, user accounts don't exist in the Kubernetes API server; instead, the API server treats user identities as `opaque` data.
  - **Service Accounts**
    - Used by machines
- When a service account is created it two objects.
  - the Service Account object
    - `kubectl describe serviceaccount <name>`
  - Secret object to store the token init.
    - `kubectl describe secret <token_name>`
- Then the Secret object is linked to the service account
- **Process**
  1.  Create a SA `kubectl create sa <SA_name>`
  2.  Export the token `kubectl create token <SA_name>`
      1. Token path: `/var/run/secrets/kubernetes.io/serviceaccount`
  3.  Assign right permissions with (**RBAC**)
      1. Create Role
      2. Create RoleBindings
  4.  Attach the `SA` to `POD` using `serviceAccountName` field
  5.  for checking permissions use below command
      1. `kubectl auth can-i get pod --as=system:serviceaccount:default:<SA_name>`
- Each `NameSpace` has its own default `service account`
- When you create a cluster, Kubernetes automatically creates a ServiceAccount object named `default` for every namespace in your cluster. it has `no permission` by default
- whenever a new pod is created the default `service Account` and its token automatically gets mounted as a volume inside the pod
- if you don't mention the `service Account` name a default gets automatically mounted as a volume mount inside the pod
- To prevent Kubernetes from automatically injecting credentials for a specified ServiceAccount or the `default` ServiceAccount, set the `automountServiceAccountToken` field in your Pod specification to `false`.
  > **Note**: After Kubernetes version `1.24` the token has expiration time

```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubernetes.io/enforce-mountable-secrets: "true"
  name: my-serviceaccount
  namespace: my-namespace
```

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader-SA
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"
```

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: pod-reader-SA
  apiGroup: rbac.authorization.k8s.io
```

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  serviceAccountName: my-serviceaccount
  containers:
  - name: nginx
    image: nginx
    command: ["sleep", "5000"]
```

To remove the `default` SA from pod

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  automountServiceAccountToken: false # To do not use the default SA
  containers:
  - name: nginx
    image: nginx
    command: ["sleep", "5000"]
```

---

## [Image Security](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

- to connect to private container registry other than docker
- image: `registry_url/user_Account/Image_repo`
  - ![alt text](image-20.png)
- to authenticate to the private docker register you need to create a secret

```
kubectl create secret docker-registry docker-creds \
--docker-server= private-registry.io \
--docker-username= registry-user \
--docker-password= registry-password \
--docker-email= registry-user@org.com
```

```yml
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry
type: kubernetes.io/dockerconfigjson
data:
  docker-server: gcr.io
  docker-username: xyz
  docker-password: password
  docker-email: xyz@mail.com
```

- Attach the `secret` to the `pod` with `imagePullSecrets` field in pod definition

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: registry_url/user_Account/Image_repo # give full image Name
    image: nginx
    command: ["sleep", "5000"]
  imagePullSecrets:
  - docker-registry
```

---

## Docker Security

- container and host share the same kernel
- All the process run by the container are ran on the host itself but in there own namespace
- Process of containers are isolated from each other
- by default the docker run process as a root user inside the container
  - you can choose the `user` while running the container ex- `user 1000`
- use can build a `Dockerfile` with `user 1000` then any process docker run inside the docker container will be run as a `user 1000`

```Dockerfile
FROM ubuntu
USER 1000
```

```
docker run my-ubuntu-image sleep 3600 # this will run the sleep command as a USER 1000 not the ROOT inside the container
```

- `ROOT` user within the container is not as good as the `ROOT` user in the host
  - Docker host doesn't have the privilege to reboot the host
  - **`docker run --cap-add MAC_ADMIN ubuntu`** : This option adds a specific capability to the container.
    - **`MAC_ADMIN`** stands for Mandatory Access Control (MAC) administration. This capability allows the container to modify or administer MAC policies. Such policies control access to system resources, enforcing security boundaries.
    - _Note_: Granting the `MAC_ADMIN` capability should be done carefully, as it gives the container additional privileges, potentially increasing security risks.
  - **`docker run --cap-drop KILL ubuntu`** : This option removes a specific capability from the container.
    - **`KILL` capability**: Normally, this capability allows processes within the container to send signals to other processes (e.g., to terminate them). By dropping it, you prevent processes within the container from sending signals to terminate other processes (unless they own the processes).
    - **Security Context**: Dropping capabilities like `KILL` is part of a security best practice called the "principle of least privilege." By reducing the container's privileges, you minimize the risk of it being used to compromise the system.
  - **`docker run --priviledge ubuntu`** : This flag grants the container almost all the host’s kernel capabilities. Specifically - The container has access to all devices on the host, allowing it to perform operations that are usually restricted. - The container can change certain settings and modify system configurations, which are typically protected (e.g., mount filesystems, change kernel parameters). - It bypasses many of Docker's usual isolation mechanisms, essentially giving the container root access to the host machine’s kernel and hardware.
    > **Note**: **Security Risk**: Running a container with the `--privileged` flag is risky because it grants nearly unrestricted access to the `host` system. This should only be used in trusted environments or when absolutely necessary, such as in cases requiring direct hardware access (e.g., for containers managing low-level system operations or running software that needs kernel-level permissions).

---

## Docker security Context in Kubernetes:

- To Implement the docker like capabilities via k8s
- to apply at `pod` level add `securityContext` field

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: ubuntu_conatiner
    image: ubuntu
    command: ["sleep", "3600"]
```

To apply capabilities at `container` level inside the pod

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: ubuntu_conatiner
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```

> **Note**: Capabilities cannot be added at the `POD` level it can be only added at the `container` level

```yml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"] # This command will run as USER 1002
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"] # This command will run as USER 1001
```

---
