### POD

- we can use `command` to override the `ENTRYPOINT` from pod defination,works like `--entrypoint`
- we can use `args` to override the `CMD` from pod defination

![alt text](image-6.png)

Example 1 of `Command` and `args`

```
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
    command: ["sleep", "5000"]
# OR
    command:
    - "sleep"
    - "5000"
```

Example 2 of `Command` and `args`

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
    command: ["sleep"]
    args: ["5000"]
```

> **Note**:
> Both the `command` and `args` need to string not number
