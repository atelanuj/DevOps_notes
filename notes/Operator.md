# Operator

- K8s Operators are software extensions that make use of Kubernetes APIs to extend behavior.
- Kubernetes allows users to automate the deployment and execution of workloads, and also the way that Kubernetes does these things.
- **Operators are patterns that extend the behavior of the cluster** without changing the Kubernetes code. Its APIs act as custom resource controllers.
- **they are controllers that execute loops to check the actual state of the cluster and the desired state**
- **Operators are software extensions that use custom resources to manage applications and their components**
- using Operators enables us to view an application as a single object that exposes only the adjustments that make sense for the application, instead of a collection of primitives (such as Pods, Deployments, Services, or ConfigMaps).
- K8s Operators are controllers for packaging, managing, and deploying applications on Kubernetes.
- the Operator uses **Custom Resources (CR)** that define the desired configuration and state of a specific application through **Custom Resource Definitions (CRD)**
- Operators are actual programs that run in the cluster and interact through Kubernetes APIs to automate more complex functions than those natively handled by Kubernetes itself
- Operators are declarative rather than imperative tools, because our role is to define the desired objectives and resources while their responsibility is to adjust the system to keep it as close to the desired state as possible
    - The ability to **deploy** an application on demand;
    - Making a **backup** of an application state or restarting an application from a given backup;
    - Managing **the update of an application** with all its dependencies including new configuration settings and necessary database changes;
    - Exposing a service to applications that do not support **Kubernetes APIs**.
- they are application-specific controllers that **extend the functionality of the Kubernetes APIs**.
- Operators make it possible to **extend Kubernetes functionality to stateful applications as well, not just stateless** applications
- **Operators are definitely complex to create from scratch.** They require programming skills (preferably **GO**, but they can be implemented in any language as client/server communications) and a thorough knowledge of Kubernetes native controllers and its operating mechanisms (reconciliation loops).

## Diagram

![image](https://www.cncf.io/wp-content/uploads/2022/07/k8s-operator-900x506.webp)

!https://www.cncf.io/wp-content/uploads/2022/07/k8s-operator-900x506.webp

## CRD

- The [CustomResourceDefinition](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) API resource allows you to define custom resources. Defining a CRD object creates a new custom resource with a name and schema that you specify. The Kubernetes API serves and handles the storage of your custom resource. The name of the CRD object itself must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) derived from the defined resource name and its API group; see [how to create a CRD](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#create-a-customresourcedefinition) for more details. Further, the name of an object whose kind/resource is defined by a CRD must also be a valid DNS subdomain name.

### **Create a CustomResourceDefinition**

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: **stable.example.com**
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
    
```

and create it:

```bash
kubectl apply -f resourcedefinition.yaml
```

Then a new namespaced RESTful API endpoint is created at:

```bash
/apis/stable.example.com/v1/namespaces/*/crontabs/...
```

### **Create custom objects**

- After the CustomResourceDefinition object has been created, you can create custom objects.
- In the following example, the `cronSpec` and `image` custom fields are set in a custom object of kind `CronTab`.
- If you save the following YAML to `my-crontab.yaml`:

```yaml
apiVersion: "stable.example.com/v1"
kind: **CronTab**
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

- and create it:

```bash
kubectl apply -f my-crontab.yaml
```

- You can then manage your CronTab objects using kubectl. For example:

```bash
kubectl get crontab
```

- Should print a list like this:

```bash
NAME                 AGE
my-new-cron-object   6s
```

```bash
kubectl get ct -o yaml
```

- You should see that it contains the custom `cronSpec` and `image` fields from the YAML you used to create it:

```yaml
apiVersion: v1
items:
- apiVersion: stable.example.com/v1
  kind: CronTab
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"stable.example.com/v1","kind":"CronTab","metadata":{"annotations":{},"name":"my-new-cron-object","namespace":"default"},"spec":{"cronSpec":"* * * * */5","image":"my-awesome-cron-image"}}        
    creationTimestamp: "2021-06-20T07:35:27Z"
    generation: 1
    name: my-new-cron-object
    namespace: default
    resourceVersion: "1326"
    uid: 9aab1d66-628e-41bb-a422-57b8b3b1f5a9
  spec:
    **cronSpec: '* * * * */5'
    image: my-awesome-cron-image**
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```
## Custom ControlLoop

## OLM (Operator LifeCycle Manager)

