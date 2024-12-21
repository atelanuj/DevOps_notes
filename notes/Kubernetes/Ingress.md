## Ingress
### points
- Ingress exposes HTTP and HTTPS routes from outside the cluster to `services` within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
- An Ingress may be configured to give Services externally-reachable URLs, `load balance traffic`, `terminate SSL / TLS`, and offer `name-based virtual hosting`.
- You may need to deploy an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) such as `ingress-nginx`.
- The name of an Ingress object must be a valid DNS subdomain name.
- Ingress frequently uses `annotations` to configure some options depending on the Ingress controller, an example of which is the `rewrite-target` annotation.
- If the ingressClassName is omitted, a default Ingress class should be defined.
- 

### Diagram
![ingress](https://kubernetes.io/docs/images/ingress.svg)

### Ingress Manifests

**Path based Routing**
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80

```

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```
**Host based Routing**
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```
### Simple fanout
### Name based virtual hosting
### TLS

```yml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```
### Load balancing