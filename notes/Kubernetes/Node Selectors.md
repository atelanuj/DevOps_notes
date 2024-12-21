# [Node Selectors](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

- Node selectors helps in scheduling the pod on specific node based on labels assigned to node and the pods
- suppose we have Three nodes namely Large, Small with labels as size = large/small.
- We can use this label information to schedule our pods accordingly.

Example :

```yml
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

## Commands

Add a label to a particular node

`kubectl label nodes <node_name> key=value`

### Limitations

> NodeSelector will not be able to help in complex senerios such as
>
> - size=large or size=small
> - size !=medium (not equal)

---