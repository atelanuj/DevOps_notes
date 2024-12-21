# [Config Map](ConfigMaps)

## What is a ConfigMap

- ConfigMap can pass the key value pair to the pod
- ConfigMap can be used to pass the configuration to the pod
- it helps in managing the environment variables in the pod defination centrally
- A ConfigMap is an API object used to store non-confidential data in key-value pairs
- Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.
- The Pod and the ConfigMap must be in the same namespace.
- ConfigMap does not provide secrecy or encryption
- A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.'
- a ConfigMap has `data` and `binaryData` fields rather than spec.
- The data field is designed to contain `UTF-8` strings while the binaryData field is designed to contain binary data as `base64-encoded` strings.
- **The Kubernetes feature `Immutable` Secrets and ConfigMaps provides an option to set individual Secrets and ConfigMaps as immutable**.
  - protects you from **accidental** (or unwanted) updates that could cause applications outages
  - **improves performance** of your cluster by significantly reducing load on kube-apiserver, by closing watches for ConfigMaps marked as immutable.

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

> **Note**: Once a ConfigMap is marked as `immutable`, it is not possible to revert this change nor to mutate the contents of the data or the binaryData field. You can only delete and recreate the ConfigMap. Because existing Pods maintain a mount point to the deleted ConfigMap, it is recommended to recreate these pods.

> **Note**: A ConfigMap is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed `1 MiB`. If you need to store settings that are larger than this limit, you may want to consider **mounting a volume** or use a separate database or file service.

### Creating a ConfigMap

- Imperative
  - Adding config map through command line
    - kubectl create configmap `configmap-name` --from-literal=`key=value` --from-literal=`key2=value2`
    - kubectl create configmap `configmap-name` --from-file=`path-to-file`
    - kubectl create configmap `configmap-name` --from-file=`app.config.properties`
- Diclarative
  - Adding through the pod defination

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

command :

- `kubectl create -f configmap.yml`
- `kubectl describe configmaps`
- `kubectl get configmaps`

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:

    1. Inside a container command and args
    2. Environment variables for a container
    3. Add a file in read-only volume, for the application to read
    4. Write code to run inside the Pod that uses the Kubernetes API to read ConfigMap

1. Using `env` in pod defination

```yml
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
    env:
      - name: USERNAME
        value: admin
      - name: APP_COLOUR
        value: Pink
```

- adding env varibles as `USERNAME = admin` and `APP_COLOUR = pink`in pod defination directly.

1. **Using `valueFrom` in pod defination**

```yml
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
    env:                      # Define the environment variable
      - name: APP_COLOUR      # Notice that the case is different here
                              # from the key name in the ConfigMap.
        valueFrom:
          configMapKeyRef:
            name: APP         # The ConfigMap this value comes from.
            key: frontend     # The key to fetch.
      - name: UI_PROPERTIES_FILE_NAME
        valueFrom:
          configMapKeyRef:
            name: game-demo                 # different configmap
            key: ui_properties_file_name    # key to fetch the value of
```

- The value of `frontend` will be assigned to `APP_COLOUR` env variable from `APP` configmap
- The value of `ui_properties_file_name` will be assigned to `UI_PROPERTIES_FILE_NAME` env variable from `game-demo` configmap

2. **Using `envFrom` in pod defination**

```yml
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
    envFrom:
      - configMapKeyRef:
          name: app-config
```

Use `envFrom` to define all of the ConfigMap's data as container environment variables. The key from the ConfigMap becomes the environment variable name in the Pod.

3. **Mounting the configMap through the `Volumes`.
   To consume a ConfigMap in a volume in a Pod:**

- Create a ConfigMap or use an existing one. Multiple Pods can reference the same ConfigMap.
- Modify your Pod definition to add a volume under `.spec.volumes[]`. Name the volume anything, and have a `.spec.volumes[].configMap.name` field set to reference your ConfigMap object.
- Add a `.spec.containers[].volumeMounts[]` to each container that needs the ConfigMap. Specify `.spec.containers[].volumeMounts[].readOnly = true` and `.spec.containers[].volumeMounts[].mountPath` to an unused directory name where you would like the ConfigMap to appear inside the container.
  - If there are multiple containers in the Pod, then each container needs its own `volumeMounts` block, but only **one** `.spec.volumes` is needed per ConfigMap
- Modify your image or command line so that the program looks for files in **that directory**. Each key in the ConfigMap data map becomes the filename under mountPath.
- These volumes can be of various types, such as `emptyDir`, `hostPath`, `persistentVolumeClaim`, `configMap`, or `secret`.
- The `volumeMounts` section allows you to connect the volumes defined in the Pod specification to specific paths within the container's filesystem
- This enables containers to access and store data persistently, share data between containers in the same Pod, or access configuration files and secrets securely.
- Mounted ConfigMaps are updated automatically

```yml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```
