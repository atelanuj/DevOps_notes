# [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

## What is a Secret

- A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.
- Using a Secret means that you don't need to include confidential data in your application code.
- Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.
- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a `tmpfs` so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

## Use Cases

- [Set environment variables for a container.](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
- [Provide credentials such as SSH keys or passwords to Pods.](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#provide-prod-test-creds)
- [Allow the kubelet to pull container images from private registries.](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

# How to create a Secret

- Imperative
  - commands
    - kubectl create secret generic `secret-name` --from-literal=`key`=`value`
    - kubectl create secret generic `secret-name` --from-literal=`key`=`value` --from-literal=`key1`=`value1`
    - kubectl create secret generic `secret-name` --from-file=`path-to-file`
    - kubectl create secret generic `app-secret` --from-file=`app-secret.properties`
- Diclarative

```yml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  DB_HOST: dmFsdWUtMg0KDQo= # this is a encoded value
  DB_USER: dmFsdWUtMQ0KDQo= # this is a encoded value
  DB_PASS: dmFsdWUtMl0KDQo= # this is a encoded value
```

To encode a values in a secret use:

```
echo -n 'my-value-to-be-encoded' | base64
```

to decode the values in a secret use:

```
echo -n 'c29tZS12YWx1ZQ==' | base64 --decode
```

1. `envFrom` injecting secret as a Environment variables

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      envFrom:
        - secretRef:
            name: dotfile-secret
```

1. `valueFrom` injecting secret as a `Single` Environment variables

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: dotfile-secret
              key: username
```

2. Mount secret from a `volume`

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
```

> **Notes**:
>
> - Secrets are not encrypted they only encoded.
> - dont push secrets to code repos
> - secrets are not encrypted in `etcd
>   - enable the encryption at REST
> - Anyone able to create pods/deploys in the same namespace can access the secrets
>   - configure less privileged access to secrets - `RBAC`
> - Consider third party secrets providers
> - AWS provider
> - AZURE provider
> - GCP provider
> - valut provider
