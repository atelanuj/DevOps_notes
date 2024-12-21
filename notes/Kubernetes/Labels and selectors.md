# [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

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

```yml
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